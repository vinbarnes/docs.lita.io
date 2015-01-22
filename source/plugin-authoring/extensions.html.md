---
guide: Plugin Authoring
section: Extensions
menu: plugin-authoring
---

%p An extension is a plugin that adds new functionality to Lita's core in order to extend the capabilities of adapters and handlers. Unlike adapters and handlers, extensions are not used in a user's Lita project directly, but are dependencies of adapter and handler gems.

%p To create a new extension plugin, run the following command in your shell: <kbd>lita extension <var>NAME_OF_YOUR_EXTENSION</var></kbd>. This generates the files necessary to build an extension plugin and publish it as a Ruby gem.

.alert.alert-info
  %strong Note:
  Extensions are an advanced topic and most of the time you won't need to write them yourself.

%h3#hooks Hooks

%p Hooks are the primary mechanism for extensions to add new functionality to Lita's core. They are similar to the event system, allowing objects to register themselves with different hook names which get called when that hook is triggered. A callback object is registered using <code>Lita.register_hook</code>. The callback object is any object that responds to <code>:call</code> and accepts a single hash of payload data as an argument.

%p The available hooks are as follows:

%table.table.table-bordered
  %tr
    %th Name
    %th Description
    %th Payload
  %tr
    %td <code>:before_run</code>
    %td Called when <code>Lita.run</code> is invoked. Useful for extensions that need to perform operations just before the robot starts.
    %td
      %ul
        %li <code>:config_path</code> (String) - The file path to the Lita config file that will be loaded, or <code>nil</code> if there is no config file.
  %tr
    %td <code>:validate_route</code>
    %td Called at the end of incoming message dispatch checks. If the the callback object returns <code>true</code>, the message will be dispatched to the route. If the callback object returns <code>false</code>, the message will fail to match the route, and the dispatcher moves on to trying the next route.
    %td
      %ul
        %li <code>:handler</code> (Lita::Handler) - The handler class the route belongs to.
        %li <code>:route</code> (Lita::Handler::Route) - The route object for the route currently being validated.
        %li <code>:message</code> (Lita::Message) - The incoming message.
        %li <code>:robot</code> (Lita::Robot) - The currently running robot.
  %tr
    %td <code>:trigger_route</code>
    %td Called when an incoming message has matched a route, just before the handler's callback method is invoked.
    %td
      %ul
        %li <code>:response</code> (Lita::Response) - The response object that will be passed to the handler.
        %li <code>:route</code> (Lita::Handler::Route) - The route object for the route that matched.

%h3#examples Examples

%p Here is an extension that prevents routes with the custom <code>:password_protected</code> option from matching incoming messages unless they contain a secret password:

%pre
  %code.ruby
    :preserve
      module Lita
        module Extensions
          class PasswordProtection
            def self.call(payload)
              route = payload[:route]
              message = payload[:message]

              if route.extensions[:password_protected] && !message.body.include?("secret")
                return false
              else
                return true
              end
            end
          end

          Lita.register_hook(:validate_route, PasswordProtection)
        end
      end

%p To take advantage of this extension, a handler would simply add the option in a route definition:

%pre
  %code.ruby
    :preserve
      route(/^dangerous/i, :dangerous_action, command: true, password_protected: true)

%p Here is an extension that adds the SHA1 hex digest of the incoming message's body to the response object passed to the handler:

%pre
  %code.ruby
    :preserve
      require "digest/sha1"

      module Lita
        module Extensions
          class MessageSHA
            def self.call(payload)
              response = payload[:response]

              response.extensions[:sha] = Digest::SHA1.hexdigest(response.message.body)
            end
          end

          Lita.register_hook(:trigger_route, MessageSHA)
        end
      end

%p To take advantage of this extension, a handler just needs to access the value added to the response object:

%pre
  %code.ruby
    :preserve
      route(/^sha/i, :sha, command: true)

      def sha(response)
        response.reply("The SHA1 of your message is: \#{response.extensions[:sha]}.")
      end

For a more detailed example that makes use of both hooks and custom route options, check out <a href="https://github.com/jimmycuadra/lita-keyword-arguments">lita-keyword-arguments</a>.

%h3#using-extensions Using extensions

%p When using an extension to add functionality to an adapter or handler you're writing, there are a few important "gotchas" to be aware of.

%p Extensions do not "automatically activate" in the same way that adapters and handlers do when Lita is started. To load an extension, add it as a dependency in your plugin's gemspec file and manually <code>require</code> it inside your plugin's code, before the plugin is defined.

%p When testing a handler with Lita's built-in RSpec extras, the <code>Lita::Registry</code> object that stores global state, including which extension hooks are registered, gets reset for each test run. This means that in order for extensions that register hooks to be active during each test run, you need to manually register them in a <code>before</code> block. If the documentation for the extension doesn't specify which hooks it registers, you may need to look at its source code to see for yourself.

%p For example, the <a href="https://github.com/jimmycuadra/lita-keyword-arguments">lita-keyword-arguments</a> extension registers a callback to the <code>:trigger_route</code> hook. In order to activate it in tests, you would add the following code to your <code>spec/spec_helper.rb</code> file:

%pre
  %code.ruby
    :preserve
      RSpec.configure do |config|
        config.before do
          registry.register_hook(:trigger_route, Lita::Extensions::KeywordArguments)
        end
      end