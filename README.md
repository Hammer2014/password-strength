Password Strength Tester
========================

[![Build Status](https://travis-ci.org/tests-always-included/password-strength.svg?branch=master)](https://travis-ci.org/tests-always-included/password-strength)

Password Strength is a library that calculates the relative strength of a password.  This is accomplished by using several techniques.  Primarily this relies on letter trigraphs, which check each set of 3 characters in a given password.  More information on the trigraph calculations [is available](data/README.md).  This also calculates the entropy bits based on Claude Shannon's technique on determining the number of bits required to represent a set of characters and multiplying it by the length of the password.  There is also a check to see if a password is contained in a list of common passwords.

There is a wonderful [demo page](http://tests-always-included.github.io/password-strength) where you can test out the technology yourself.


Usage
-----

The Password Strength library is wrapped with [fid-umd], and is usable in a variety of systems.  When you use the `.check()` method for the password "abcd1234", the results may look like what you see below.  Comments are added to explain the fields.

    {
        charsetSize: 36,  // Explained better below
        commonPassword: true,  // If true, don't use this password!
        passwordLength: 8,  // Same as string.length
        shannonEntropyBits: 24,  // Claude Shannon's method
        strengthCode: 'WEAK',  // Our ranking of the password's strength
        trigraphEntropyBits: 39.71755017780513,  // Based on trigraphs
        charsets: {
            number: true,  // Contains 0-9
            lower: true,  // Contains a-z
            upper: false,  // Contains A-Z
            punctuation: false,  // Contains common sentence punctuation
            symbol: false,  // Contains mathematical symbols
            other: ''  // Unicode and uncaught characters
        }
    }

The charset size is the sum of the lengths of the different charsets that the password uses.  The higher this number, the harder it is to brute force attack.  That's precisely why password policies often say "must contain one lowercase letter, one uppercase letter, one number and a symbol".

The strength code is based on the trigraph entropy bits when they are available and will fall back to the Shannon entropy bits.  It's one of five values:  `VERY_WEAK`, `WEAK`, `REASONABLE`, `STRONG`, and `VERY_STRONG`.

Trigraph entropy bits is discussed more [here](data/README.md).

The charsets is mostly an object that has boolean values, except the `other` property.  That one is a catch-all string of the letters that were not caught and tallied into another one of the charset lists.  The list of characters in `other` is deduplicated.


### Browser

Include `lib/password-strength.js` in your project.

    <script src="path/to/password-strength.js" />

Next you will want to instantiate the modules and make some AJAX calls to fetch additional data and make the password strength tester more accurate and informative.  This bit of code uses jQuery, but similar code can be written for any framework.

    $(function () {
        // Create the instance that can be used immediately for strength tests
        window.passwordStrength = new PasswordStrength();

        // Add additional files that improve the results
        $.getJSON("path/to/data/common-passwords.json", function (data) {
            window.passwordStrength.addCommonPasswords(data);
        });
        $.getJSON("path/to/data/trigraphs.json", function (data) {
            window.passwordStrength.addCommonPasswords(data);
        });
    });

Later, to calculate the strength of a password you would use something like this:

    if (window.passwordStrength) {
        strength = window.passwordStrength.check("abcd1234");

        if (strength.strengthCode.indexOf('WEAK') >= 0) {
            alert("Your password is too weak.");
        }
    }


### Node.js

First run `npm install --save tai-password-strength` and then your code would look a bit like this:

    var taiPasswordStrength = require("tai-password-strength")
    var strengthTester = new taiPasswordStrength.PasswordStrength();
    var results = strengthTester.check("abcd1234");

    // Add in extra files for additional checks and better results
    strengthTester.addCommonPasswords(taiPasswordStrength.commonPasswords);
    strengthTester.addTrigraphMap(taiPasswordStrength.trigraphs);
    var betterResults = strengthTester.check("abcd1234");

    if (betterResults.strengthCode.indexOf('WEAK') >= 0) {
        throw new Error("Your password is too weak");
    }


[fid-umd]: https://github.com/fidian/fid-umd
