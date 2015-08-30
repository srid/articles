% Understanding Elm
%
% May 24, 2015

[Elm](http://elm-lang.org/) is an unusual programming language as one first needs to learn to think in a different paradigm ([FRP](https://en.wikipedia.org/wiki/Functional_reactive_programming)). To me, one of the challenges in such a process was to gain sufficient understanding of how tasks and ports are related to each other. This post was written in response to somebody on the [Elm mailing list](https://groups.google.com/d/msg/elm-discuss/BR-f9qXbt3k/npRXCdYI7woJ) being interested in my "own summary of tasks / ports / signals / mailboxes."

## Thinking in Dataflow

Part of developing an idea in Elm involves understanding the *[dataflow](https://en.wikipedia.org/wiki/Dataflow_programming)* of your program. In normal imperative programming (say in Python or Java) one essentially constructs a series of operations to perform (thus "imperative"). Whereas in Elm, you define the *dataflow*. There is no sequence of logical steps to define, hence the paradigm is *fundamentally* different.

The architectural abstractions entailed in constructing such a dataflow are: models, actions and view. The **model** describes the *current state* of your entire program at any point in time; outside of the model, there is no state and everything is ephemeral. **Actions** happen *to* the model. Acting upon a model *updates* it, resulting in a newer model; this is how your program handles state. Finally, the **view** defines how to render any model on the screen. For more information, see [the Elm architecture tutorial](https://github.com/evancz/elm-architecture-tutorial).

## Thinking in Time

Some people believe that the past and the future are *illusory*, and that only the "present" exists. Notwithstanding what one's philosophical position on time is, it is incontrovertible that the past – when it happens – always happens *now*, and the future – when it also happens – always happens *now*. As such now is all that exists. This "now," if you are keeping up with our brief philosophical foray, is essentially what the **model** refers to. There is *only one* model at any point in time; or, to state it differently but consistently, *current model* is all that exists.

Just as actions happen in the real world, changing the ever-changing "now", so too do **actions** happen in an Elm program, changing the every-changing "model".

As the model changes, by virtue of being acted upon by actions, the view changes.

As a program is, ultimately, what an user sees ("view") and interacts with ("actions") - any program can be implemented using the aforementioned model-action-view abstraction.

## Signal

How does Elm represent time?

As we have seen before, time is not an axis with discrete past, present and future; rather time is nothing more than a convenience to help us think in terms of the *changing world*. Or, *changing model*. Something changes (now) due to actions happening (also now), and this leads to the illusion of the time existing over some continuum (in reality, there is no axis, but only an ever-lasting point).

Changing model.

A better to question ask, therefore, is: how does Elm represent change (of something)?

A **signal** represents something that changes. The changing world, or changing model. A signal has a value, and that is whatever that "something" is *now*. When actions happen (now), thus acting upon on that "something" (now), it changes to "something else" (now) ... and the signal has that "something else" as its value (now).

A signal is an arena in which something, but only that thing, changes. Yet the signal itself is constant.

The key thing to remember here is that a signal is, ultimately, no special than regular values or variables used in other programming languages. In other words, in Elm, you may pass signals to functions, return them from functions or do just about anything you can do with regular types. For example, you can "map" over a signal resulting in a new signal with the underlying changing values being applied with the mapping function, and this is no different to mapping over lists.

## Signal of Model

We noted that when actions happen *to* a model, the model changes. And we know that signals represent change. So, if a model is capable of changing, how would we represent such a changing model? You guessed it!

Signal of model.

We thus have a signal of model. Although the value of this signal is whatever the model is *now*, it can change as actions happen to them.

## Tasks

If there is no state outside of model — specifically a signal of model (which is equally immutable) — how does Elm represent tasks with side-effects outside the Elm world?

Wrong question.

Tasks have side-effects *only when actually performed*. So, Elm represents tasks without actually performing them. To Elm, a "task" is nothing more than a recipe or instructions to do something (which in turn would have the side-effects).

An example task would be fetching something over the network. Wrong (are you paying attention?). An example task would be the instructions to fetch something over the network. Good.

## Performing tasks

Since tasks are instructed to have side-effects, you *cannot* perform them in the Elm world, where signals are the only model of change. Tasks have to be performed *outside* of the Elm world; specifically they have to be performed in the JavaScript runtime (outside of Elm). Before we get to talking about performing tasks, however, we must understand ports.

## Ports

A **port** is a communication hub between the Elm world (with its models, actions and views) and the outside world. Something from the outside world can *send* values to the Elm world through a port. Generally ports are defined to be a signal of something. This makes sense because a port — when treated as a passage with values passing one after another — is something that changes over time, and signals are designed to represent the exact abstraction.

Just as the outside world can *send* values to the Elm world through a port (with Elm seeing it as a signal of those values), the reverse too can happen. You can have the Elm world "communicate" with the outside world. Communicate? How? Wouldn't the act of "communicating" violate the aforementioned *dataflow* architecture? Indeed it would, but only in the normal sense of that word. Remember, there are no series of logical steps to do as in an imperative program. We define the dataflow — the flow of time, or changing models; and how to render them — that's it, nothing more. So how do we "communicate" with the outside world?

The problem is with the world "communicate," a verb that indicates doing something explicitly. What if we replace that with a noun? Read "something," maybe? Have the outside world read "something" from the Elm world? That is effectively the same operation, isn't it? The elm world can prepare this value to be "read" (as part of its dataflow model) and the outside world can simply read it? Now make this a signal. A signal is something that changes over time, and if this signal is defined as a port, then the outside world automatically gets a steady stream of something that changes over time. Thus, the "communication" problem is solved.

When you reflect on it all - regardless of what a port is actually used for, from the Elm side things are pretty simple: all we have are signals.

## What performs the tasks?

As show above, Elm cannot perform tasks. However we can tell the JavaScript runtime to do so. We use a port (the communication hub) to do this. Since Elm can't "communicate" (in the normal sense of the world) with the outside world, we take the aforementioned indirect route of simply *defining* the signal of whatever it is that that will be communicated.

We define a port as signal of tasks.

It seems that the Elm runtime already manages ports that are signals of tasks by effectfully peforming the tasks without any extra input from the programmer. So defining such a port is all that you need to do. The Elm runtime will take care of the rest.

## Results of tasks

A task when performed will result in something. Either an error, or a value computed (indeed Elm tasks belong to the type class `Task error value`). The programmer becomes responsible for having the task explicitly send these results back to the Elm world. Before we look into doing this, however, we need to understand mailboxes.

## Mailbox

A **mailbox** is a signal with an address value. This address value is used to *create a task to* send values to the signal. Note that the address value *cannot* be used to send values to the signal. There is a distinction here. Sending something to a signal is an imperative act, and thus can only be modeled as a task. The important thing to remember here is that mailboxes provide us with a signal that some arbitrary task can latter address.

## Sending the results of tasks

If we define a port as a signal of tasks, and if a task value of this signal is performed by the outside world, how to have the results back into the Elm world? The answer is a little trickly. While you cannot do this directly, what you can do is compose two tasks together to run one after another. The second task will take the results of the first task and send it to the address of the mailbox defined in the Elm world. Something from the Elm world which uses this mailbox signal automatically becomes aware of it.

## Are you still thinking in dataflow?

In the Elm world there is only dataflow. A simple data flow is as follows:

    {USER} <- map view <- Signal Model <- update action model <- [Signal Model, Signal Action]

 You have the signal of model and signal of action to begin with. There is an update function that takes an action and a model, returning a new changed model. The "main" routine consequently does this update by fold'ing over the action signal; the result being another signal of models. The same main routine may compose the resulting signal of models by mapping the view function over it. This results in signal of whatever the view produces, and if the view produces HTML we have a signal of HTML values. The final piece of the dataflow puzzle involves the Elm runtime taking these signal of HTML values and updating the browser DOM (that which the user sees and interacts with).

What if you had several ports and signals? Some of those ports may be ports of tasks to be run, and some of those signals may be signals of the results from the tasks. Would that affect our basic dataflow? Fundamentally, there won't be a change. You would still have your "main" routine, but this time you would be composing all those signals together and somehow reducing them to a *single* signal of models to have the view function map over. After all, a dataflow is an acyclic directed graph (DAG) ... and here, the Elm dataflow DAG has to *end* with a signal of renderable values for the user to see, and it doesn't matter how many signals the DAG starts with, much less where their values original from (Elm or the outside world).

(If you enjoyed reading this post, you may be interested in checking out [an ambitious project](https://github.com/srid/chronicle) I'm working on to improve human happiness.)
