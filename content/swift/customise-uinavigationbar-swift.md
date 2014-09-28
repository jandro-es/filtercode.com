Title: Customise UINavigationBar in Swift
Date: 2014-09-28 10:20
Category: swift
Tags: swift, ui, components
Slug: customise-uinavigationbar-swift
Authors: jandro_es
Summary: How to customise an **UINavigationBar** in iOS8 using **Swift**.

Customising an UINavigationBar using **Swift**, as with everything, is a little different to what we're used to. In our *ViewController* we need to recover the current UINavigationBar:

~~~~{.language-swift}
	var navBar = navigationController?.navigationBar
~~~~

**navigationController** is set as optional, as with many **Cocoa** properties, mainly because they can be **nil**.

let's change our *navbar* background color:

~~~~{.language-swift}
	navBar?.barTintColor = UIColor(red: 200, green: 200, blue: 200, alpha: 1.0)
~~~~

let's change our title font, font color and add a shadow:

~~~~{.language-swift}
	var barShadow: NSShadow = NSShadow()
    barShadow.shadowColor = UIColor(red: 0, green: 0, blue: 0, alpha: 0.8)
    barShadow.shadowOffset = CGSize(width: 0, height: 1)
    
    let navBarTitleTextAttributes = [NSFontAttributeName: UIFont(name: "Electrolize", size: 24.0),
        NSForegroundColorAttributeName: UIColor(red: 255, green: 255, blue: 255, alpha: 1),
        NSShadowAttributeName: barShadow
    ]
    navBar?.titleTextAttributes = navBarTitleTextAttributes
~~~~

this way we can customise all values of our current UINavigationBar, we can check all the properties in [Apple's documentation](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UINavigationBar_Class/index.html).

