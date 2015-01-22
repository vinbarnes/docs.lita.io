---
guide: Getting Started
section: Plugins
menu: getting-started
---

%p The core Lita gem provides a framework to build on, but includes only a minimal amount of functionality to Lita at runtime. To make real use of it, you'll want to install an adapter plugin to allow Lita to connect to the chat service of your choice, and handler plugins to add new behavior.

%p The <a href="https://www.lita.io/plugins">plugins</a> page is a catalog of all Lita plugins, both adapters and handlers, that have been published to RubyGems. Use the catalog to find an adapter for the chat service you want to use, and as many handlers as you want for added functionality.

%p Adding the plugins you choose to Lita is as simple as adding them to the Gemfile and running the <kbd>bundle</kbd> command in your shell to update the Gemfile.lock. The <kbd>bundle</kbd> command comes from Bundler, which is installed alongside Lita.

%p For example, to run Lita on HipChat with the karma and Google Images handlers, your complete Gemfile would look like this:

%pre
  %code.ruby
    :preserve
      source "https://rubygems.org"

      gem "lita"

      gem "lita-hipchat"

      gem "lita-karma"
      gem "lita-google-images"

%p Adapters will likely require some configuration to be able to connect. See the documentation for the adapter for details. Without installing an adapter plugin, you can still use the default shell adapter to chat with Lita in your terminal.