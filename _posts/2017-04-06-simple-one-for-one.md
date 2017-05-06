---
layout:     post
title:      "Erlang Simple One For One Examples"
date:       2017-04-06 00:00:00
summary:    Things I learned from SurfPings
categories: Erlang OTP SurfPings
---

As I was programming [SurfPings](http://surfpings.com), I needed to be regularly creating dynamic processes with simple-one-for-one supervision. Examples of this include:

1. a new process for every db query
2. a new process for every API call
3. a new process for every email sent

Unfortunately, I quickly realized there's not a lot of great documentation out there as far as examples. This post is designed to remedy that, and provide example implementations of common use cases.

The **most important thing to note**: Erlang Processes stop when they have no more code to execute. For a `gen_server`, this will occur after returning a `{stop, ...}` tuple. This means you don't have to ever terminate your children. Just `stop` and you're done!

## Use Case #1: Non-blocking, no return value required

When to use: background jobs! Set something going, don't care if it succeeds, but don't leave processes hanging around after they're done. Key point to implement this model is that the API function must init the child process, then use a `gen_server:cast`, which **must return** a `{stop, Reason, NewState}` tuple. If `{noreply, ...}` is used the child will not be removed from the supervisor tree when it's done running!

Caller:
{% highlight erlang %}
caller_function() ->
  child_process:do_something(),
  % non-blocked logic happens here
  ok.
{% endhighlight %}

Process:
{% highlight erlang %}
handle_cast(Request, State) ->
  % logic
  {stop, normal, State}

do_something() ->
  supervisor:start_child(process_sup, [Arguments]),
  gen_server:cast(Pid, {get_emails, RegionId}).
{% endhighlight %}

## Use Case #2: Blocking, no return value required

When to use: when you need to know if something completed successfully, but don't care about anything beyond that. For example, in SurfPings, this is used when clients subscribe. Given that we don't need a return value, we can actually handle all the logic in the `init` block of the child process, and just return `{stop, normal}` when we're done.

Caller:
{% highlight erlang %}
handle(<<"POST">>, <<"/api/subscribe">>, Req0) ->
  % ...
  mongo_handler:add_email(Email, Region, Threshold, WithinPeriod),
{% endhighlight %}

Process:
{% highlight erlang %}
add_email(Email, Region, Threshold, WithinPeriod) ->
  supervisor:start_child(mongo_handler_sup, [{add_email, Email, Region, Threshold, WithinPeriod}]).

init({add_email, Email, Region, Threshold, WithinPeriod}) ->
  {ok, DB} = establish_connection(),
  add_email_internal(DB, Email, Region, Threshold, WithinPeriod),
  {stop, normal}.
{% endhighlight %}

You could also use `gen_server:call` after initializing the child process. See Use Case #3.

## Use Case #3: Blocking, return value required

When to use: whenever you need to get a value synchrnously, but want to have the callee be in its own process for purposes of resiliency or architecture. In SurfPings, this is used when we retrieve emails. While this COULD be handled entirely within the `init` block, by returning `{stop, {data could go here]}`, this would be pretty poor practice and require co-opting `gen_server terminate/2` in order to get your return values. Instead, we start a child, and issue a `gen_server:call` to it. The return value is then returned from the child process via `{stop, Reason, Response, State}`.

Caller:
{% highlight erlang %}
get_emails_for_region(RegionId) ->
  {ok, Pid} = supervisor:start_child(mongo_handler_sup, [{get_emails, RegionId}]),
  % note: passing parameters here is not necessary: all logic could be handled in the `handle_call`.
  gen_server:call(Pid, {get_emails, RegionId}).
{% endhighlight %}

Process:
{% highlight erlang %}
init(_Args) ->
  % any state creating logic goes here (optional)
  {ok, State}.

handle_call({get_emails, RegionId}, _From, DB) ->
  EmailsForRegion = get_emails_internal(RegionId, DB),
  {stop, normal, EmailsForRegion, DB}.
{% endhighlight %}

## Use Case #4: Non-blocking, return value required (later)

When to use: when you want to run jobs in parallel as background tasks, but need to use the values they return. This is the `Promise.all()` of Erlang. For example, we could issue a DB query and an API query simultaneously, and then send emails after both child processes have completed.

This requires no additional work as far as the child process, which will look exactly equivalent to Use Case #3. However, the caller will need to yield its `gen_server` with `rpc:nb_yield`.

Alternate strategy: it is possible to use `gen_server:cast()`, manually sending a response, if you were to pass a `FromPID` as part of the message. Not as clean, not really recommended
