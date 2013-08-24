---
layout: post
title: "Stop selenium loading firefox in the foreground"
date: 2013-08-24 15:57
comments: true
categories: 
- BDD
- TDD
- RSpec
- Capybara
- Selenium webdriver
published: true
---

In my day job, an important part of our testing suite involves driving Firefox via RSpec feature specs.
Which is realised through the use of the [Capybara](https://rubygems.org/gems/capybara) and [Selenium Webdriver](https://rubygems.org/gems/selenium-webdriver) gems.

As all testing suites grow, we eventually introduced the awesome [Parallel Tests](https://rubygems.org/gems/parallel_tests) gem, cutting run-times dramatically.
An unfortunate side effect however, multiple instances of Firefox randomly opening in the foreground and stealing focus.

Some digging into the selenium-webdriver codebase found the culprit in the **Selenium::WebDriver::FireFox::Launcher** class:

{% codeblock [selenium-webdriver]/lib/selenium/webdriver/firefox/launcher.rb:64 lang:ruby %}
def start
  assert_profile
  @binary.start_with @profile, @profile_dir, "-foreground"
end
{% endcodeblock %}

Removing the **-foreground** parameter has the desired effect of loading Firefox behind the terminal, and not stealing focus.
But since the selenium-webdriver codebase is hosted in google code a pull request was out of the question, time for some monkey patching.

The following code goes at the base of spec_helper:

{% codeblock [your-project]/spec/spec_helper.rb lang:ruby %}
if ENV.fetch('SELENIUM_WEBDRIVER_FIREFOX_NO_FOREGROUND', '').downcase == 'true'
  module Selenium::WebDriver::Firefox
    class Launcher
      def start
        assert_profile
        @binary.start_with @profile, @profile_dir, ''
      end
    end
  end
end
{% endcodeblock %}

The patch is disabled by default, to opt-in set the environment variable as part of your testing command:

{% codeblock lang:bash %}
SELENIUM_WEBDRIVER_FIREFOX_NO_FOREGROUND=true rake parallel:spec
{% endcodeblock %}

Happy testing folks!
