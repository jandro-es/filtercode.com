Title: Functional tests in iOS with Appium
Date: 2015-07-15 10:20
Category: swift
Tags: swift, testing, appium, python, selenium
Slug: testing-swift-appium
Authors: jandro_es
Summary: Functional testing using **Appium** and it´s *Selenium* driver in iOS with Python.
Status: draft

[**Appium**](http://appium.io) it's a system for multi platform automated testing for mobile apps. Even with XCode 7's new UI testing feature, using Appium has many advantages, like being platform agnostic and really easy integration with CI systems like Jenkins. In case your company has a QA team, it allows them to write tests using almost any programming language. In fact any progamming language with a **Selenium** driver, which almost any QA team will know how to use it.

You can write tests for Appium in *Java*, *Javascript*, *Ruby*, *Objective-C*, *PHP*, *Python*, etc.. In this post we will use **Python**. You can execute your tests in Simulators or in real devices, we´ll show how to do both.

First we need to install in our computer the following packages:

- HomeBrew
- NodeJS
- npm
- Appium
- Appium python client
- ideviceinstaller (optional)

###Install HomeBrew, node and npm

For a correct installation that allows you to install node packages globally without using sudo (required by appium) you can see this [**post**](http://www.filtercode.com/devops/npm-global-without-sudo).


###Install Appium, Appium client library and ideviceinstaller

For installing Appium we will use **npm**

~~~~{.language-bash}
	npm install -g appium
~~~~

now we need the **python appium client library** for installing it, as with many python packages we will use **pip*:

~~~~{.language-bash}
	pip install Appium-Python-Client
~~~~

At this moment you´ll need to install a diferent client library if you intend to write your tests in a different language.

**IMPORTANT** as for all python projects, we'll recommend to use **virtualenv** & **pip freeze** to have diferent environments.

For **ideviceinstaller** we´ll use **brew**:

~~~~{.language-bash}
	brew install ideviceinstaller
~~~~

This packages is required if we intent to execute the tests in a physical devive, not only in the simulator.

At this point we should have a working testing environment. Let's write out first test.

###Our first Appium test

For our first test, we're going to test a simple form application. The application will have two *UITextFields* and two *UIButtons*. One button will clear the text property of the *UITextFields* and the other one will "*submit*" the contents of the *UITextFields*. For this example, the submission will just display an *UIAlertViewController* with a *String* containing the *submitted* values.

