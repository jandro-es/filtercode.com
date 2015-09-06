Title: Functional tests in iOS with Appium and Python
Date: 2015-09-05 10:20
Category: swift
Tags: swift, testing, appium, python, selenium
Slug: testing-swift-appium
Authors: jandro_es
Summary: Functional testing using **Appium** and it´s *Selenium* driver in iOS with Python.

[**Appium**](http://appium.io) it's a system for multi platform automated testing for mobile apps. Even with XCode 7's new UI testing features, use Appium has many advantages:

- **Platform independent testing system**. You can use similar tests and infrastructure for testing iOS or Android applications.
- **Integration with not Apple CI systems**. Yes, you can use XCode Server for CI, but if your company already have CI and build systems for other platforms/technologies you most likely will need to use **Jenkins**, **Travis CI** and similar.
- **Uses Selenium Web Driver**. Which is a common use technology for testing, so a lot of people knows how to use it.

You can write tests for Appium in *Java*, *Javascript*, *Ruby*, *Objective-C*, *PHP*, *Python*, etc.. In this post we will use **Python**. You can execute your tests in Simulators or in real devices, we´ll show how to do both.

First we need to install in our computer the following packages:

- HomeBrew
- NodeJS
- npm
- Appium
- Appium python client
- ideviceinstaller (optional)

###Install HomeBrew, node and npm

For a installation that'll allow you to install node packages globally without using sudo (required by appium) you can see this [**post**](http://www.filtercode.com/devops/npm-global-without-sudo).


###Install Appium, Appium client library and ideviceinstaller

To install Appium we'll use **npm**

~~~~{.language-bash}
	npm install -g appium
~~~~

now we need the **python appium client library**, to install it, as with many python packages we will use **pip*:

~~~~{.language-bash}
	pip install Appium-Python-Client
~~~~

if you intend to use a different programming language to write the tests, you'll need to install the matching client library.

**IMPORTANT** as for all python projects, the normal way of work is to use **virtualenv** & **pip freeze** to have diferent environments.

For **ideviceinstaller** we install it using **brew**:

~~~~{.language-bash}
	brew install ideviceinstaller
~~~~

This package is required if you plan to execute tests in a physical device.

At this point you should have a working testing environment. Let's write our first test.

###Our first Appium test

For our first test, we're going to test a simple form application. The application will have two *UITextFields* and two *UIButtons*. One button will clear the text property of the *UITextFields* and the other one will "*submit*" the contents of the *UITextFields*. For this example, we're only going to test the clearing of the fields to check that every action and outlet are properly set.

We're going to write a test using **Appium**, it will test that the app behaviour is as defined before. In a normal **BDD** or **TDD* workflow you'll write the test before the app or feature itself and you'll make the app *pass* the tests afterwards. In this case and for easy of use we already have the app already working. You can download it from [**here**](https://github.com/jandro-es/TestAppium)

We create a *Python* file called **basic_test.py**, next we declare our class:

~~~~{.language-python}
	#!/usr/bin/python
	#  -*- coding: utf-8 -*-

	import unittest
	import string
	import random

	from appium import webdriver

	class BasicTest(unittest.TestCase):

	if __name__ == '__main__':
    	suite = unittest.TestLoader().loadTestsFromTestCase(BasicTest)
    	unittest.TextTestRunner(verbosity=2).run(suite)
~~~~

Our *BasicText* class must inherit from **unittest.TestCase** and import **webdriver** from the appium module. Then we configure the verbosity in the main function and we're ready to write our test cases and configure our test environment. As mentioned before, Appium can execute tests on the simulator or in physical devices. We create a **setup** method that will configure the environment, for a physical device is like this:

~~~~{.language-python}
	def setUp(self):
        bundle_id = "<app_bundle_id>"
        device_name = "<device_name>"
        udid = "<udid_device>"
        self.driver = webdriver.Remote(
            command_executor='http://127.0.0.1:4723/wd/hub',
            desired_capabilities={
                'bundleId': bundle_id,
                'platformName': 'iOS',
                'deviceName': device_name,
                'udid': udid 
            })
~~~~

you need to replace the following values:

- **app_bundle_id** for the bundle id of the app you want to test.
- **device_name** for the name of the device you'll use for testing as it appears on the **device** window in **XCode**.
- **udid_device** for the **UDID** of the device as you see it in **iTunes**.

If you want to execute your tests in the simulator instead of in a device, the setup would be like this:

~~~~{.language-python}
	def setUp(self):
        app = os.path.join(os.path.dirname(__file__),
                           '<path_to_simulator_build>',
                           '<app_name>')
        app = os.path.abspath(app)
        self.driver = webdriver.Remote(
            command_executor='http://127.0.0.1:4723/wd/hub',
            desired_capabilities={
                'app': app,
                'platformName': 'iOS',
                'platformVersion': '8.4',
                'deviceName': 'iPhone 6'
            })
~~~~

where **path_to_simulator_build** is the path to the build folder inside you app's package, something like this:

	../../apps/TestAppium/build/release-iphonesimulator

and where **app_name** is the name of the app, but not the *display* name, the real app name, which is something like: 
	
	TestAppium.app

Like most of test suites we have the **tearDown** method to restore state:

~~~~{.language-python}
	def tearDown(self):
        self.driver.quit()
~~~~

The flow of the test should be something like this:

- Locate the textfields.
- Populate the textfields with random data so the result is not dependant in any way from the input.
- Locate the *clear* button and click it.
- Get the new value of the textfields and check that they are empty.

which will result in code similar to this:

~~~~{.language-python}
	def test_ui_clearing(self)::
        self._populate_random_textfields(5, ['TextField1', 'TextField2'])
        self.driver.find_element_by_name('ClearButton').click()

        textfield1_value = self.driver.find_element_by_name('TextField1').text
        textfield2_value = self.driver.find_element_by_name('TextField2').text

        self.assertEqual(textfield1_value, "")
        self.assertEqual(textfield2_value, "")
~~~~

we are using these two helper functions to generate random strings:

~~~~{.language-python}
	def _random_string_length(self, length):
        """
        Generates a random string of the given length.
        :param length: the length of the desired string.
        :return: the generated string.
        """
        return ''.join(random.SystemRandom().choice(string.lowercase) for _ in range(length))

    def _populate_random_textfields(self, string_length, textfields):
        """
        Internal method to populate the given textfields with random strings of the given length.
        :param string_length: the length of the generated string.
        :param textfields: a list of textfield names.
        :return: a dictionary with the generated values
        """
        values = {}

        for textfield_name in textfields:
            textfield = self.driver.find_element_by_name(textfield_name)
            values[textfield_name] = self._random_string_length(string_length)
            textfield.send_keys(values[textfield_name])

        return values
~~~~

Now we can run our test suite to check if the app clears the textfield correctly. We need to start the **Appium** server in a separate process:

~~~~{.language-bash}
	appium &
~~~~

now we execute our test suite:

~~~~{.language-bash}
	python basic_test.py
~~~~

if all the tests passed successfully we´ll see an output like this:

~~~~{.language-bash}
	test_ui_clearing (__main__.BasicTest) ... ok

	----------------------------------------------------------------------
	Ran 1 test in 28.864s

	OK
~~~~

in case some of the tests failed, you´ll see messages in the like of this:

~~~~{.language-bash}
	test_ui_clearing (__main__.BasicTest) ... ERROR

	======================================================================
	ERROR: test_ui_clearing (__main__.BasicTest)
	----------------------------------------------------------------------
	Traceback (most recent call last):
	  File "basic_test.py", line 102, in test_ui_clearing
	    self._populate_random_textfields(5, ['TextField3', 'TextField2'])
	  File "basic_test.py", line 88, in _populate_random_textfields
	    textfield = self.driver.find_element_by_name(textfield_name)
	 .....
	NoSuchElementException: Message: An element could not be located on the page using the given search parameters.


	----------------------------------------------------------------------
	Ran 1 test in 10.171s

	FAILED (errors=1)
~~~~

this basic test shows the boilerplate to create more advanced and even multi platform tests using Appium. If you're not confortable using *python* you can easily switch to any of the supported languages.

###Notes

These cheats will help you save time when creating and using **Appium** tests:

- Make sure you device has **Enable UI automation** active inside *Settings -> Developers*.
- Disable third party keyboards if you have any in the device. They make the input driver go haywire.
- Install the app in the device before running the tests.
- To find an element by name, the driver uses the **accessibilityLabel** of the UIControl.
- If you're using real devices for testing, remember to uninstall the previous app before installing the new one. This is really important when you're changing *accessibilityLabels*.

**You can find the test application and the test itself in this [Github repository](https://github.com/jandro-es/TestAppium).**
