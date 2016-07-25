---
layout: post
title: "My first Alexa skill"
description: "How to get going with building simple Alexa skills in python"
category:
tags: ['amazon', 'echo', 'alexa', 'lambda', 'skill', 'python']
---

I wrote this a while ago as a gist for those of us playing with the office Echo and finally decided to clean it up for my blog. The documentation is pretty good - have a read of the [getting started guide](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/getting-started-guide)!

## Skill components

A skill is a set of intents (actions) and the metadata to be able to map voice data to those intents. They are made up of 3 pieces:

 * Examples of how an intent is used in a sentence. Think of this as a bit like a `urlconf` in Django - only you have multiple of them for the same view.
 * Intent schemas that describes what actions your skill can do and the parameters an action takes. Crucially they not only say `MyIntent` takes a parameter of `MyParameter` but also that it is of type `AMAZON.NUMBER` - this is how Alexa knows what words you might use in the parameter part of a sentence.
 * A backend that is called with an intent and its parameters.

### Backend

You need to run some code somewhere. There are 2 mechanisms Amazon supports - once is to post to an HTTPS endpoint and one is to invoke an AWS Lambda function. For my testing I used the [alexandra](https://pypi.python.org/pypi/alexandra) python package. It abstracts away dealing with HTTP or Lambda and you just use decorators to map Alexa intents to python functions. Here is my `test.py`:

```python
import alexandra

app = alexandra.Application()
name_map = {}

@app.launch
def launch_handler(session):
    return alexandra.reprompt('What would you like to do?')

@app.intent('MyNameIs')
def set_name_intent(slots, session):
    name = slots['Name']
    name_map[session.user_id] = name

    return alexandra.respond("Okay, I won't forget you, %s" % name)

@app.intent('WhoAmI')
def get_name_intent(slots, session):
    name = name_map.get(session.user_id)

    if name:
        return alexandra.respond('You are %s, of course!' % name)

    return alexandra.reprompt("We haven't met yet! What's your name?")

if __name__ == '__main__':
    app.run('0.0.0.0', 8080, debug=True)
```

Assuming you have an active virtualenv with `alexandra` installed you can run this from a terminal with just:

```bash
python test.py
```

I use ngrok to expose a public https url that the skill can use:

```bash
ngrok http 8080
```


### Intents

When you define example utterances you use placeholders to tell Alexa where the variables are to capture. These are slots and they are defined and attached to intents in a JSON schema:

```json
{
    "intents": [
        {
            "slots": [
                {
                    "type": "AMAZON.LITERAL",
                    "name": "Name"
                }
            ],
            "intent": "MyNameIs"
        },
        {
            "slots": [],
            "intent": "WhoAmI"
        }
    ]
}
```

The default slot types that are available are:

* `AMAZON.DATE`
* `AMAZON.DURATION`
* `AMAZON.FOUR_DIGIT_NUMBER`
* `AMAZON.NUMBER`
* `AMAZON.TIME`
* `AMAZON.US_CITY`
* `AMAZON.US_FIRST_NAME`
* `AMAZON.US_STATE`
* `AMAZON.LITERAL`

See available [Slot Types](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/alexa-skills-kit-interaction-model-reference#Slot%20Types).

You can define your own custom types in the web console, and these can extend the built-in types. A custom type is just the name of your type and a list of up to 50,000 possible values. These are converted automatically into a spoken form and an output form, so the value that is sent to your skill might be slightly different to the one you defined - e.g. `2 beers` instead of `two beers`.

In our example we used `AMAZON.LITERAL`. As you will see, that means we have to give lots of examples of that literal in our utterances. This slot type is actually deprecated and we should use `AMAZON.US_FIRST_NAME` or our own custom type.

There are built-in intents for when you are building sessions. For example, if you needed to ask for confirmation before taking an action you can use the `AMAZON.YesIntent`. See [Built-in Intents](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/implementing-the-built-in-intents#Available%20Built-in%20Intents).


### Utterances

Then we need our sentence data. These are just examples of how a user might say an intent.

```
MyNameIs call me {john|Name}
MyNameIs call me {frank|Name}
MyNameIs call me {stacey|Name}
MyNameIs call me {helen|Name}
WhoAmI say my name
WhoAmI tell me name
```

You need to design as many possible utterance variations as you can. Your utterances do not become top level sentences that Alexa responds to. For example, you don't say "Alexa, say my name" you actually say "Alexa, ask myskill to say my name". See [Beginning a conversation](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/supported-phrases-to-begin-a-conversation).

At the time of writing, there is no 'best guess at what they said' slot type. In this example, if you say "Alexa, ask myskill to call me Henrietta" it will think you said Helen.

In the example above we used the `AMAZON.LITERAL` slot type. If we had used the `AMAZON.US_FIRST_NAME` slot type then the utterances would simply be:

```
MyNameIs call me {Name}
WhoAmI say my name
WhoAmI tell me name
```

## Wiring it all together

This is a bit clunky. As yet there isn't any deployment tooling, we have to set up the skill manually in a browser:

* Sign up and sign in to [developer.amazon.com](http://developer.amazon.com/)
* Choose "Alexa", choose "Alexa Skills Kit/Get Started" then "Add New Skill"
* Name your skill and give it an invocation word. If you chose "frank" then in the example above you would say "Alexa, ask frank to call me stacey".
* In the next window you can copy and paste the intent schema we defined.
* If we used any custom slot types this is where we would define them.
* Also on the same screen you can copy and paste your sample utterances.
* Click next. You can point the skill at the https url you got from `ngrok` or to a lambda function you deployed.
* On the next screen if using HTTPS you will have to tell it some more information about your certificate. This is mostly to enable self-signed certificates for testing which is not needed with `ngrok`.
* The next screen is the test screen. This screen is a little weird because it is half way through a wizard type interface. But actually your skill is saved and you can now test it on an Echo linked to your developer.amazon.com account. You can also use this page to directly interface with your backend using the Service Simulator - no Echo required, just type what you would have said.
* After that you are on your own - the final launch steps for your skill which I haven't tried yet.
