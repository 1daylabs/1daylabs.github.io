---
title: "How to bypass encryption mechanism in Android apps"
date: 2020-08-24 00:00:00 +0530
categories: [Android]
tags: [android,frida,crypto] 
image: "assets/img/Posts/BypassAndroidSecurity/thumb.jpeg"
---
*Original artwork by raywenderlich.com*

Hi Folks, hope you are well. As you know developers and pentesters are always into a cat and mouse game. No matter how much we want to deny the fact but we make each other's life a little tough. Mobile application developers these days implement an encryption mechanism in their applications to prevent tampering. 

However, there are ways to bypass such mechanisms. I am going to demonstrate a scenario which will help you when you come across such application behavior.

We are going to use Injured Android for the demonstration. [Injured Android](https://github.com/B3nac/InjuredAndroid) is a CTF style vulnerable application built to practice android testing. Huge shoutout to the author [B3nac](https://twitter.com/b3nac) for building this application.

## Why is it worth reading?

I have come across many applications that rely highly on client-side encryption, as an extra layer of security for us to uncover. When you intercept the traffic using your favorite proxy tool, all you'll see is encrypted text. It is because all of the requests are being encrypted on the client-side and sent to the server. The server will have the same encryption algorithm coded, so upon receiving the data, it will decrypt and fetch the values.

If you are into Android testing or want to get started, this article is for you.

## Installation

Download the latest build from [here](https://github.com/B3nac/InjuredAndroid/releases). I am using [genymotion](https://www.genymotion.com/) for this article. Get the virtual device rolling using genymotion and install the APK by entering the following command.

`adb install injuredandroid.apk`

After the successful installation, you'll see the following screen.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-03-56.png "Home screen")

## Identification

We are interested in **FLAG SIX - LOGIN 3**. This is how our FlagSix screen looks like, it has a single textbox and a **SUBMIT** button.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-15-09.png "Flag six activity")

I have tried entering random text and clicked on submit but nothing happened. Our next step is to look at the java source code in order to understand the functionality of this activity. 

Let's decompile the APK to fetch the application source code. I used [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) for the sake of simplicity, it makes the task a lot easier. Below is the synopsis of our APK from MobSF.


![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-35-40.png "MobSF output")

There are a whole bunch of [activities](https://www.tutorialspoint.com/android/android_acitivities.htm) to look at. **FlagSixLoginActivity** seems to be our destination.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-40-40.png "Activity List")

## May the source(code) be with you

Let's take a look at the code. Focus on `submitFlag()` Method.

```java
package b3nac.injuredandroid;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import androidx.appcompat.app.c;
import androidx.appcompat.widget.Toolbar;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;
import d.m.b.d;

public final class FlagSixLoginActivity extends c {
    private int t;

    static final class a implements View.OnClickListener {

        /* renamed from: c  reason: collision with root package name */
        final /* synthetic */ FlagSixLoginActivity f1979c;

        a(FlagSixLoginActivity flagSixLoginActivity) {
            this.f1979c = flagSixLoginActivity;
        }

        public final void onClick(View view) {
            if (this.f1979c.F() == 0) {
                if (view != null) {
                    Snackbar X = Snackbar.X(view, "Keys.", 0);
                    X.Y("Action", (View.OnClickListener) null);
                    X.N();
                    FlagSixLoginActivity flagSixLoginActivity = this.f1979c;
                    flagSixLoginActivity.G(flagSixLoginActivity.F() + 1);
                    return;
                }
                d.f();
                throw null;
            } else if (this.f1979c.F() != 1) {
            } else {
                if (view != null) {
                    Snackbar X2 = Snackbar.X(view, "Classes.", 0);
                    X2.Y("Action", (View.OnClickListener) null);
                    X2.N();
                    this.f1979c.G(0);
                    return;
                }
                d.f();
                throw null;
            }
        }
    }

    public final int F() {
        return this.t;
    }

    public final void G(int i) {
        this.t = i;
    }

    /* access modifiers changed from: protected */
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView((int) R.layout.activity_flag_six_login);
        i.g.a(this);
        C((Toolbar) findViewById(R.id.toolbar));
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new a(this));
    }

    public final void submitFlag(View view) {
        EditText editText = (EditText) findViewById(R.id.editText3);
        d.b(editText, "editText3");
        if (d.a(editText.getText().toString(), j.a("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ=="))) {
            Intent intent = new Intent(this, FlagOneSuccess.class);
            FlagsOverview.D = true;
            new i().b(this, "flagSixButtonColor", true);
            startActivity(intent);
        }
    }
}
```

Here, we can clearly see **submitFlag** method which will be called on click of **SUBMIT** button. It takes value from the text box(editText3), converts it to string and compares it with the output of `j.a("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ==")` method. `k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ==` is our encrypted text.

This application has some level of obfuscation used, we need to identify what `j.a()` method is? We have `b3nac/injuredandroid/j.java` class in our package.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-16%2018-28-19.png "j. java")

`a()` method has the following code, it is a decryption method. We can clearly figure out from the code that [DES](https://en.wikipedia.org/wiki/Data_Encryption_Standard) symmetric-key algorithm is used. **Congrats first milestone achieved!**

The code below uses the value from `f2002a` to generate the secret key.

```java
    public static String a(String str) {
        if (b(str)) {
            try {
                SecretKey generateSecret = SecretKeyFactory.getInstance("DES").generateSecret(new DESKeySpec(f2002a));
                byte[] decode = Base64.decode(str, 0);
                Cipher instance = Cipher.getInstance("DES");
                instance.init(2, generateSecret);
                String str2 = new String(instance.doFinal(decode));
                Log.d("Oh snap!", "Decrypted: " + str + " -> " + str2);
                return str2;
            } catch (InvalidKeyException | NoSuchAlgorithmException | InvalidKeySpecException | BadPaddingException | IllegalBlockSizeException | NoSuchPaddingException e2) {
                e2.printStackTrace();
            }
        } else {
            System.out.println("Not a string!");
            return str;
        }
    }
```

Our next goal is to find the key, it must be coded somewhere. **Expedition continues..**

Class `j` has a byte array defined which takes value from `h.a()` method.

```private static final byte[] f2002a = h.a();```

### h.java has the following code

```java
package b3nac.injuredandroid;

import android.util.Base64;

public class h {

    /* renamed from: a  reason: collision with root package name */
    private static byte[] f1997a = Base64.decode("Q2FwdHVyM1RoMXM=", 0);

    /* renamed from: b  reason: collision with root package name */
    private static String f1998b = "9EEADi^^:?;FC652?5C@:5]7:C632D6:@]4@>^DB=:E6];D@?";

    static {
        Base64.decode("e0NhcHR1cjNUaDFzVG9vfQ==", 0);
    }

    static byte[] a() {
        return f1997a;
    }

    static String b() {
        return f1998b;
    }
}
```

Method `a()`  returns a byte array of the key, but before that it is decoding a base64 string `Q2FwdHVyM1RoMXM=`

Let's decode it to find out the actual key.

`echo Q2FwdHVyM1RoMXM= | base64 -d`

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-16%2018-57-04.png "Secret key")

### Second milestone achieved

We got the key: `Captur3Th1s`

Now this key can be used to decrypt the ciphertext using some custom java class or online tool. But we are going for a much simpler option.

### Let's take a pause here

While testing an application there are some keywords that will quickly help us in identifying classes, algorithm used for the encryption. Although it can be little tricky, when the code is obfuscated.

#### Keywords

encrypt, decrypt, crypt, AES, DES, SecretKeyFactory, secretKey, Cipher, InvalidKeyException
etc.

Let's search these keyword in the current APK to see if we actually land on `j.java` file.

#### DES

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-23%2000-25-52.png "DES")

#### SecretKeyFactory

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-23%2000-29-17.png "SecretKeyFactory")

#### InvalidKeyException

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-23%2000-31-05.png "InvalidKeyException")

This is how we can quickly identify the classes responsible for encryption. 

### Back to our example

We have the decryption method at our disposal. Now as per our understanding, the application is taking user input and comparing it with the decrypted value of `k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ==`.

What if, we can see what the decrypted value of aforementioned ciphertext is, when the application is comparing the value? Yes!! we can definitely do that!

To accomplish this task we are going to use [FRIDA](https://frida.re/), it is a Dynamic instrumentation toolkit. Using Frida we can modify application logic on run-time, it allows us to inject our scripts into running processes. It is a must have toolkit in your android testing arsenal.

**To summarize:** We'll hook the decryption method `a()` of class `j` and when the application attempts to compare the strings, we'll print the plain text value.

### Frida on the roll

Let's have a quick look at the following JS code which will invoke the decryption method `a()` with our supplied value. We have built this JS code to override the decryption method.

#### decrypt.js

```js
console.log("Script loaded successfully ");

Java.perform(function x() {
    console.log("Inside java perform function");
    var my_class = Java.use("b3nac.injuredandroid.j"); // class responsible for decryption
    
    var string_class = Java.use("java.lang.String");

    my_class.a.overload("java.lang.String").implementation = function (x) { //hooking the new function
        console.log("*************************************")
        var my_string = string_class.$new("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ=="); // cipher text
        console.log("Original arg: " + x);
        var ret = this.a(my_string);
        console.log("Return value: " + ret);
        console.log("*************************************")
        return ret;
    };
});
```

Method `a()` requires only one argument, the cipher text to decrypt. We have supplied the cipher text in the `this.a()` method it'll store the response in string variable `ret`.

#### Time to run the Frida script.

`frida -U -f b3nac.injuredadnroid --no-pause -l decrypt.js`

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-22%2020-19-09.jpg "Frida invoke")

As we can see, Frida script is running successfully. This will start the application in our emulator.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-03-56.png "Home screen")

Frida script has overridden the decryption method. We'll see the effect of this when we interact with the flag6 activity. So let's go to flag six activity. Enter any value and click on **SUBMIT** button.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-22%2020-31-11.png "Hooked")

There is no activity happening on the screen but if we go to our Frida console we'll have the following output along with the decrypted text `{This_Isn't_Where_I_Parked_My_Car}`

By overriding the decryption method we have successfully decrypted the cipher text to plain text.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-22%2020-34-54.png "Done")

Now imagine, if this was a real application then you can use the same technique to inject your payloads before it gets encrypted and sent to server. This way we have bypassed the encryption logic written in the application to make the app tamper-free.

I enjoyed writing this article and hope that you enjoyed reading it.

**Happy hacking :-)**


## References

* https://github.com/B3nac/InjuredAndroid
* https://github.com/MobSF/Mobile-Security-Framework-MobSF
* https://frida.re/docs/android/


Author - Jaimin Gohel

Twitter handle : [@jaimin_gohel](https://twitter.com/jaimin_gohel)
</br>
LinkedIn handle : [jaimin-gohel-440a4a52](https://www.linkedin.com/in/jaimin-gohel-440a4a52)
