Title: Logger utility class in Swift for iOS and OSX
Date: 2015-02-15 10:20
Category: swift
Tags: swift, frameworks, utility, bluebird
Slug: logger-swift
Authors: jandro_es
Summary: A logger class in **Swift** log level control, coloured output and some advanced features.

When an application reaches certain size, the amount of debug messages can make the console nearly unusable. Thus making the process of debugging the app really hard. The idea behind this class, is to have a logger mechanism similar to *LogCat* in Android (of course without a lot of features) while keeping the simplicity of a *println* statement. The code is as follows:


~~~~{.language-swift}
	//
	//  Logger.swift
	//
	//  Created by Alejandro Barros Cuetos on 03/02/15.
	//  Copyright (c) 2015 Alejandro Barros Cuetos. All rights reserved.
	//
	//  Redistribution and use in source and binary forms, with or without
	//  modification, are permitted provided that the following conditions are met:
	//
	//  1. Redistributions of source code must retain the above copyright notice, this
	//  list of conditions and the following disclaimer.
	//
	//  2. Redistributions in binary form must reproduce the above copyright notice,
	//  this list of conditions and the following disclaimer in the documentation
	//  && and/or other materials provided with the distribution.
	//
	//  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
	//  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
	//  IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
	//  ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
	//  LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
	//  CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
	//  SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
	//  INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
	//  CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
	//  ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
	//  POSSIBILITY OF SUCH DAMAGE.
	//

	import Foundation

	/**
	Available Log level for Logger
	- None:    Print no message
	- Error:   Message of level Error
	- Warning: Message of level Warning
	- Success: Message of level Success
	- Info:    Message of level Info
	- Custom:  Message of level Custom
	*/
	enum LoggerLevels: Int {
	    
	    case None = 0
	    case Error
	    case Warning
	    case Success
	    case Info
	    case Custom
	}

	/**
	*  Singleton class to print custom log messages easier to read
	*  and analize. Based in glyphs and log levels
	*/
	class Logger {

	    // MARK: - Properties

	    var verbosityLevel: LoggerLevels = .Custom

	    var errorGlyph: String = "\u{1F6AB}"    // Glyph for messages of level .Error
	    var warningGlyph: String = "\u{1F514}"  // Glyph for messages of level .Warning
	    var successGlyph: String = "\u{2705}"   // Glyph for messages of level .Success
	    var infoGlyph: String = "\u{1F535}"     // Glyph for messages of level .Info
	    var customGlyph: String = "\u{1F536}"   // Glyph for messages of level .Custom

	    // MARK: Public methods

	    /**
	    Prints a formatted message through the debug console,
	    showing a glyph based on the loglevel and the name of the file
	    invoking it if present
	    
	    :param: message      Message to print
	    :param: logLevel     Level of the log message
	    :param: file         Implicit parameter, file calling the method
	    :param: line         Implicit parameter, line which the call was made
	    */
	    func logMessage(message: StaticString , _ logLevel: LoggerLevels = .Info, file: StaticString = __FILE__, line: UWord = __LINE__) {
	        
	        if self.verbosityLevel.rawValue > LoggerLevels.None.rawValue &&  logLevel.rawValue <= self.verbosityLevel.rawValue {
	            println("\(getGlyphForLogLevel(logLevel))\(message) [\(file):\(line)] \(message)")
	        }
	    }
	    
	    /**
	    Prints a formatted message through the debug console,
	    showing a glyph based on the loglevel and the name of the class
	    invoking it if present
	    
	    :param: message      Message to print
	    :param: logLevel     Level of the log message
	    :param: file         Implicit parameter, file calling the method
	    :param: line         Implicit parameter, line which the call was made
	    
	    :returns: A formatted string
	    */
	    func getMessage(message: StaticString, _ logLevel: LoggerLevels = .Info, file: StaticString = __FILE__, line: UWord = __LINE__) -> String {
	        
	        return("\(getGlyphForLogLevel(logLevel))\(message) [\(file):\(line)] \(message)")
	    }
	    
	    /**
	    Logs the given message as a custom message and
	    check the condition of the assert.
	    
	    :param: condition Condition clousure for the assert
	    :param: message   Message to print
	    :param: file      Implicit parameter, file calling the method
	    :param: line      Implicit parameter, line which the call was made
	    */
	    func logMessageAndAssert(condition: @autoclosure () -> Bool, _ message: StaticString, file: StaticString = __FILE__, line: UWord = __LINE__) {
	        
	        logMessage(message, .Custom)
	        assert(condition, message.description, file: file, line: line)
	    }
	    
	    // MARK: - Private Methods
	    
	    /**
	    Returns the Glyph to use according to the passed LogLevel
	    
	    :param: logLevel
	    
	    :returns: A formatted string with the matching glyph
	    */
	    private func getGlyphForLogLevel(logLevel: LoggerLevels) -> String {
	        
	        switch logLevel
	        {
	        case .Error:
	            return "\(errorGlyph) "
	        case .Warning:
	            return "\(warningGlyph) "
	        case .Success:
	            return "\(successGlyph) "
	        case .Info:
	            return "\(infoGlyph) "
	        case .Custom:
	            return "\(customGlyph) "
	        default:
	            return " "
	        }
	    }

	    // MARK: - Singleton Pattern

	    class var sharedInstance: Logger {

	        struct Singleton {
	            static let instance = Logger()
	        }

	        return Singleton.instance
	    }
	}

~~~~

First we have an **enum** with the available log levels:

~~~~{.language-swift}
	enum LoggerLevels: Int {

        case None = 0
        case Error
        case Warning
        case Success
        case Info
        case Custom
    }
~~~~

With them, we can set up the **vervosityLevel** of the Logger to the level we desire each time:

~~~~{.language-swift}
	Logger.sharedInstance.vervosityLevel = .Warning
~~~~

this way, the Logger will only print the messages with a level equal to **Warning** or inferior (being in this case only Warning and Error). We can have a clearer console output when we want to trace a problem or a verbose one when not.

For even clearer messages, the Logger, by default (you can change them easily) prefixes the output with **unicode** glyphs for each log level:

~~~~{.language-swift}
	var errorGlyph: String = "\u{1F6AB}"    // Glyph for messages of level .Error
    var warningGlyph: String = "\u{1F514}"  // Glyph for messages of level .Warning
    var successGlyph: String = "\u{2705}"   // Glyph for messages of level .Success
    var infoGlyph: String = "\u{1F535}"     // Glyph for messages of level .Info
    var customGlyph: String = "\u{1F536}"   // Glyph for messages of level .Custom
~~~~

It will print the filename which logged the message and the line number inside the file. To accomplish this easily we implicity pass this data using **Swift's built-in identifiers** as the method signature shows:

~~~~{.language-swift}
	func logMessage(message: StaticString , _ logLevel: LoggerLevels = .Info, file: StaticString = __FILE__, line: UWord = __LINE__)
~~~~

The available methods for the Logger are:

~~~~{.language-swift}
	func logMessage(message: StaticString , _ logLevel: LoggerLevels = .Info, file: StaticString = __FILE__, line: UWord = __LINE__)

	func getMessage(message: StaticString, _ logLevel: LoggerLevels = .Info, file: StaticString = __FILE__, line: UWord = __LINE__) -> String

	func logMessageAndAssert(condition: @autoclosure () -> Bool, _ message: StaticString, file: StaticString = __FILE__, line: UWord = __LINE__)
~~~~

- The method **logMessage** will print the message into the console, similar to a **println** command but with the enhanced features.
- The method **getMessage** will format a String in the same way, but instead of printing it, it will return it.
- The method **logMessageAndAssert** will print the message using **logMessage** and then will evaluate the condition.

Examples:

~~~~{.language-swift}
	Logger.sharedInstance.logMessage("Testing Error Message", .Error)
        
    Logger.sharedInstance.logMessage("Testing Warning Message", .Warning)
    
    Logger.sharedInstance.logMessage("Testing Success Message", .Success)
    
    Logger.sharedInstance.logMessage("Testing Info Message", .Info)
    
    Logger.sharedInstance.logMessage("Testing Custom Message without calling class", .Custom)

    Logger.sharedInstance.getMessage("Index out of bounds", .Error)

    Logger.sharedInstance.logMessageAndAssert({3 > 5}(), "Index out of bounds")
~~~~

The code for the Logger (part of an utilities framework written in Swift for iOS and OSX under active development) is available in [Github](https://github.com/jandro-es/Bluebird/blob/master/Bluebird/src/Logger.swift)
