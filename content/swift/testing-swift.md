Title: Functional tests in iOS with Appium
Date: 2015-07-03 10:20
Category: swift
Tags: swift, testing, appium, python, selenium
Slug: testing-swift
Authors: jandro_es
Summary: Functional testing using **Appium** and it´s *Selenium* driver in iOS with Python.
Status: draft

[**Appium**](http://appium.io) it's a system for multi platform automated testing for mobile apps. Even with XCode 7's new UI testing feature, using Appium has many advantages, like being platform agnostic and really easy integration with CI systems like Jenkins. In case your company has a QA team, it allows them to write tests using almost any programming language. In fact any progamming language with a **Selenium** driver, which almost any QA team will know how to use it.

You can write tests for Appium in *Java*, *Javascript*, *Ruby*, *Objective-C*, *PHP*, *Python*, etc.. In this post we will use **Python**. You can execute your tests in Simulators or in real devices, we´ll show both.

First we need to install in our computer the following software:

- HomeBrew
- NodeJS
- npm
- Appium
- ideviceinstaller
- Python

##Install HomeBrew

I'm sure most of you already have *brew* on your systems, in case you don´t this line will download it for you:

~~~~{.language-ruby}
	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~~