# Elixir-Slack

This is a work in progress Slack [Real Time Messaging API] client for Elixir.
You'll need a Slack API token which can be retrieved from the [Web API page] or
by creating a new [bot integration].

[Real time Messaging API]: https://api.slack.com/rtm
[Web API page]: https://api.slack.com/web
[bot integration]: https://my.slack.com/services/new/bot

## Usage

Add Slack to your `mix.exs` `application` and `dependencies` methods. You'll
also need [websocket_client] since hex.pm doesn't install git based dependencies.

[websocket_client]: https://github.com/jeremyong/websocket_client

```elixir
def application do
  [applications: [:logger, :slack]]
end

def deps do
  [{:slack, "~> 0.0.5"},
   {:websocket_client, git: "https://github.com/jeremyong/websocket_client"}]
end
```

Define a module that uses the Slack behaviour and defines the appropriate
callback methods.

```elixir
defmodule SlackRtm do
  use Slack

  def start_link(initial_state) do
    Slack.start_link(__MODULE__, "token_value", initial_state)
  end

  def init(initial_state, slack) do
    IO.puts "Connected as #{slack.me.name}"
    {:ok, initial_state}
  end

  def handle_message({:type, "message", response}, slack, state) do
    state = state ++ [response.text]

    message = "Received #{length(state)} messages so far!"
    Slack.send(message, response.channel, slack)

    {:ok, state}
  end

  def handle_message({:type, type, _response}, _slack, state) do
    {:ok, state}
  end
end
```

You can send messages to channels using `Slack.send` which takes the message as
the first argument, channel as the second, and the `slack` argument as the
third.

`:websocket_client.send({:text, "Hello!"}, slack)`.

You can also access the properties about the user you're authenticated as by
calling `me` on the passed in `slack` state. More details about what's passed as
`me` can be found on the Slack [rtm.start documentation page][rtm.start].

[rtm.start]: https://api.slack.com/methods/rtm.start

Slack has *a lot* of message types so it's a good idea to define a callback like
above where unhandled message types don't crash your application. You can find a
list of message types and examples on the [RTM API page].

You can find more detailed documentation on the [Slack hexdocs page].

[RTM API page]: https://api.slack.com/rtm
[Slack hexdocs page]: http://hexdocs.pm/slack/
