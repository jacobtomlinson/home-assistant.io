---
layout: page
title: "Alexa / Amazon Echo"
description: "Instructions how to connect Alexa/Amazon Echo to Home Assistant."
date: 2015-12-13 13:02
sidebar: true
comments: false
sharing: true
footer: true
logo: amazon-echo.png
ha_category: Voice
featured: true
---

There are two ways that you can use Amazon Echo and Home Assistant together.

No matter which method(s) you decide to use, please remember that Amazon Echo requires an active Internet connection to function. If your Internet is down or experiencing issues (or Amazon's infrastructure is having issues), neither of these methods will work.

Amazon has released [Echosim], a website that simulates the Alexa service in your browser. That way it is easy to test your skills without having access to a physical Amazon Echo.

[Echosim]: https://echosim.io/

### {% linkable_title I just want to turn devices on and off using Echo %}

If you just want to be able to turn anything with a switch (like lights, switches, media players, etc) on and off, check out Michael Auchter's [Haaska][haaska-github-link] which integrates the [Alexa Lighting API][alexa-lighting-api] into Home Assistant.

[haaska-github-link]: https://github.com/auchter/haaska
[alexa-lighting-api]: https://developer.amazon.com/public/binaries/content/assets/html/alexa-lighting-api.html

Implementing Haaska means you can turn things on and off by simply saying

> Alexa, turn the living room lights on.

or

> Alexa, set the living room lights to twenty percent.

instead of

> Alexa, tell Home Assistant to turn the living room lights on.

or

> Alexa, tell Home Assistant to set the living room lights to twenty percent.

In addition, you would need to build custom intents for each device and on/off combination using the below method, whereas everything just works without any extra work by using Haaska.

Please note that you can use Haaska and the built-in Alexa component side-by-side without issue if you wish.

### {% linkable_title I want to build custom commands to use with Echo %}

The built-in Alexa component allows you to integrate Home Assistant into Alexa/Amazon Echo. This component will allow you to query information and call services within Home Assistant by using your voice. Home Assistant offers no built-in sentences but offers a framework for you to define your own.

<div class='videoWrapper'>
<iframe width="560" height="315" src="https://www.youtube.com/embed/1Ke3mtWd_cQ" frameborder="0" allowfullscreen></iframe>
</div>

#### {% linkable_title Requirements before using %}
Amazon requires the endpoint of a skill to be hosted via SSL. Self-signed certificates are ok because our skills will only run in development mode. Read more on [our blog][blog-lets-encrypt] about how to set up encryption for Home Assistant. If you are unable to get HTTPS up and running, consider using [this AWS Lambda proxy for Alexa skills](https://forums.developer.amazon.com/forums/thread.jspa?messageID=18604).

[blog-lets-encrypt]: https://home-assistant.io/blog/2015/12/13/setup-encryption-using-lets-encrypt/

To get started with Alexa skills:

 - Log in to [Amazon developer console](https://developer.amazon.com)
 - Go to Apps & Services => Alexa => Alexa Skill Kit - Get Started
 - Add a new skill
   - Name: Home Assistant
   - Invocation name: home assistant (or be creative, up to you)
   - Version: 1.0
   - Endpoint:
     - https
     - https://YOUR_HOST/api/alexa?api_password=YOUR_API_PASSWORD

#### {% linkable_title Configuring your Amazon Alexa skill %}

Alexa works based on intents. Each intent has a name and variable slots. For example, a `LocateIntent` with a slot that contains a `User`. Example intent schema:

```json
{
  "intents": [
    {
      "intent": "LocateIntent",
      "slots": [
      {
          "name": "User",
          "type": "AMAZON.US_FIRST_NAME"
        }]
    },
    {
      "intent": "WhereAreWeIntent",
      "slots": []
    }
  ]
}
```

To bind these intents to sentences said by users you define utterances. Example utterances can look like this:

```text
LocateIntent Where is {User}
LocateIntent Where's {User}
LocateIntent Where {User} is
LocateIntent Where did {User} go

WhereAreWeIntent where we are
```

This means that we can now ask Alexa things like:

 - Alexa, ask Home Assistant where Paul is
 - Alexa, ask Home Assistant where we are

### {% linkable_title Configuring Home Assistant %}

Out of the box, the component will do nothing. You have to teach it about all intents you want it to answer to. The way it works is that the answer for each intent is based on [templates] that you define. Each template will have access to the existing states via the `states` variable but will also have access to all variables defined in the intent.

You can use [templates] for the values of `speech/text`, `card/title` and `card/content`.

Actions are using the [Home Assistant Script Syntax] and also have access to the variables from the intent.

[templates]: /topics/templating/
[Home Assistant Script Syntax]: /getting-started/scripts/

Configuring the Alexa component for the above intents would look like this:

```yaml
{% raw %}
# Example configuration.yaml entry
alexa:
  intents:
    WhereAreWeIntent:
      speech:
        type: plaintext
        text: >
          {%- if is_state('device_tracker.paulus', 'home') and
                 is_state('device_tracker.anne_therese', 'home') -%}
            You are both home, you silly
          {%- else -%}
            Anne Therese is at {{ states("device_tracker.anne_therese") }} and
            Paulus is at {{ states("device_tracker.paulus") }}
          {% endif %}

    LocateIntent:
      action:
        service: notify.notify
        data_template:
          message: The location of {{ User }} has been queried via Alexa.
      speech:
        type: plaintext
        text: >
          {%- for state in states.device_tracker -%}
            {%- if state.name.lower() == User.lower() -%}
              {{ state.name }} is at {{ state.state }}
            {%- endif -%}
          {%- else -%}
            I am sorry, I do not know where {{ User }} is.
          {%- endfor -%}
      card:
        type: simple
        title: Sample title
        content: Some more content{% endraw %}
```
