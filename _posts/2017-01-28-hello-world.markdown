---
layout: post
title:  "Disco Mate"
date:   2017-01-28 18:15:08 +1100
categories: jekyll update
---

### Welcome to the disco!

Program your own disco mate! Most likely you are familiar with the high tech gadgets such as Google's [Siri][siri] or Amazon's [Alexa][alexa]. In this tutorial, you will be guided through the making of a simplified personal assistant running on the [32L476GDISCOVERY] [disco] ('disco') board. 

![image]({{ site.url }}/image/board2.jpg)

Before we jump in and get started, I'll fill you in on some personal motivations for this project. At the end of 2016, I spent a month at the [Beijing Institute of Technology][BIT] learning about the [Internet of Things] [IOT] (IOT).

The IOT refers to the internetworking of physical devices, vehicles, buildings and other items which are embedded with software, sensors and network connectivity so that they can connect and exchange data. 
More and more devices are continually being incorporated into the IOT and cutting edge research is happening rapidly in related areas such as Big Data, Cloud Computing, and Machine Learning. If you haven't explored this arena already, I definitely encourage you to take a look! 

![image](/images/IOT.jpg )


Being exposed to the current progress in IOT made me realise that we are not far off a world where personal robots could suggest a suitable outfit based on the day's weather forecast, instruct you on when to leave for work based on the current traffic, and recommend a music playlist to match your mood based on the sentiment of your text messages. As we go through our daily lives, we expect technology to provide us with updated information, emotional support, and constant entertainment. The fruits of such technological advancements are many, however there are also new challenges emerging surrounding issues such as memory and privacy on low powered sensor devices. 

While it is becoming more common for low powered sensor devices to have internet connectivity, unfortunately our particular disco board does not. Not to worry! We can think of this 'disco' mate project as a first trial at making a lightweight personal assistant.  Maybe in the future this mate can be upgraded to a fancy internet-connected device (potential promotion for a coming tutorial)! One plus is that the disco board we are using is [mbed][mbed] enabled, allowing us to use an operating system and developer ecosystem specially tailored for IOT and low powered devices. 

For this tutorial, I will assume you have a working disco board and, if you haven't done so already, you have set up an account and workspace using the mbed online compiler. This [link] [compiler] will get you started and show you what documentation already exists. 

This tutorial is targeted at computer students comfortable with programming but not necessarily amazing C++ coders. The main assumption I will make is that the you are a confident, self-driven learner who will try to fill in gaps and consult other sources when needed. While completing this tutorial, it might be useful to have your favourite C++ manuals or websites handy. If you want some personal recommendations here are two: [cplusplus][cplus-plus] and [learncpp][learn]. 

To get started, we are going to `create new program` in the workspace. Double check that your platform is `DISCO-L476VG` and choose the template `Basic examples showing how to drive the 2 LEDs`. Give your program an appropriate name such as `DiscoMate`. 

![image](/images/duplicate.jpg )

If you haven't already seen the template we have just chosen, take sometime to have a look at it and compile the code. When are the LED lights on the boards turning on and off? Try pressing different buttons on the joystick. 

We haven't mentioned how we are going to make this disco mate work. Maybe looking at the board's features you can come up with some ideas. Without giving everything away, lets say that the two most important components are the joystick and the LCD screen. 

[IOT]:http://www.forbes.com/sites/jacobmorgan/2014/05/13/simple-explanation-internet-things-that-anyone-can-understand/#2041b1f06828
[BIT]:https://english.bit.edu.cn/
[cplus-plus]: http://www.cplusplus.com/
[disco]: http://www.st.com/en/evaluation-tools/32l476gdiscovery.html
[mbed]: https://www.mbed.com/en/
[compiler]: https://developer.mbed.org/handbook/mbed-Compiler
[siri]: http://www.apple.com/au/ios/siri/ 
[alexa]:https://www.amazon.com/Amazon-Echo-Bluetooth-Speaker-with-WiFi-Alexa/dp/B00X4WHP5E
[learn]: http://www.learncpp.com/

### message CENTRE

Normally the first thing we expect a friend (human or robot) to say to us when we greet them is `Hello`.  Lets get our disco mate to say Hello! To use the LCD screen, we need to add the libraries `LCD_DISCO_L476VG` and `BSP_DISCO_L476VG` to our program. One way is to copy them over from the template titled `Basic example showing how to drive the Glass LCD`. Make some additions to the top of your main program. 
{% highlight c++ %}
#include "mbed.h"
#include "LCD_DISCO_L476VG.h"

LCD_DISCO_L476VG lcd;{% endhighlight %}



In the `main` function, we want to add some lines of code to make something appear on the LCD display. The function `lcd.DisplayString()` looks like a good option! Check out the header and source file of `LCD_DISCO_L476VG` library to learn more about this function and other functions related to the LCD display. After you have added the lines of code below, compile and run the program on the board. Does it do what you expected?  

{% highlight c++ %}
uint8_t stemp[7] = {0};

int main() {
	sprintf((char *)stemp, "HELLO");
	lcd.DisplayString(stemp);  
{% endhighlight %}

Now, say we weren't satisfied with that welcome message. Perhaps we want our disco mate to greet us in another language. Let's make it so the `CENTRE` button of the joystick provides us with a new greeting every time it is pressed.

To do this, it will be useful to learn some more about the functionality of the joystick. Observe that the top of the template we are using includes reference to the `InterruptIn` class. Take a look at the `mbed` library and the `InterruptIn` class documentation. 

{% highlight c++ %}
// Joystick button
InterruptIn center(JOYSTICK_CENTER);
InterruptIn left(JOYSTICK_LEFT);
InterruptIn right(JOYSTICK_RIGHT);
InterruptIn up(JOYSTICK_UP);
InterruptIn down(JOYSTICK_DOWN); 
{% endhighlight %}

In the `InterruptIn` class documentation can you understand the role of the `rise` and `fall` functions? 
These appear near the bottom of our template in the following lines of code. Take some time to understand how the physical movement of the joystick is connected to these functions. You may need to go back and look through the libraries and relevant files again. Out of the functions named below, which one is relevant to our immediate task?

{% highlight c++ %}
// Both rise and fall edges generate an interrupt
center.fall(&center_released);
center.rise(&center_pressed);
left.fall(&left_released);
left.rise(&left_pressed);
right.fall(&right_released);
right.rise(&right_pressed);
up.fall(&up_released);
up.rise(&up_pressed);
down.fall(&down_released);
down.rise(&down_pressed);
{% endhighlight %}

If you said `center_pressed()` you would be correct! The template already has definitions for all the functions listed above which dictate some functionality of the LED lights. We are going to modify these to suit our project (incorporating joystick and LCD display functionality). Recall that we want to have a new greeting each time the centre button is pressed. Let us define a counter (`count`) that is initialised to 0. Additionally, we are going to use some more functions from the `LCD_DISCO_L476VG` library. Add the code below and compile. Does it do what you expected? 

{% highlight c++ %}
uint32_t count = 0;
void center_pressed() {
    if (count == 1) 
    {
        lcd.Clear();
        uint8_t scroll[]= "     Yuwa";
        lcd.ScrollSentence(scroll, 2, 200);
        lcd.Clear();
    }; {% endhighlight %}

Can you write similar pieces of code (choosing your favourite languages) for `count == 2`, `count == 3`, `count == 4`? Try to change how many times and how fast the message scrolls. 

Say we decide that four different greetings is enough. What does finishing off the code with the lines below do? 
  
  {% highlight c++ %}	count++;
	if (count > 4) count = 0;
	wait(0.5);  
}  {% endhighlight %}


### UP date
Okay, so we have our disco mate saying hello to us (in possibly multiple languages). We expect most robot helpers to also be able to provide us with some useful information such as the date. How about we make the `UP` button tell us the date? 

Take some time to have a look at the C++ `time.h` library. Investigate the many different [format specifiers][time].  What is the current [unix epoch time][epoch]? 

Can you use what you have learnt in the section above to modify the function `up_pressed` so that it displays the integer day, day of the week, month, year (or any combination of these)? Perhaps you want the information to be static or to scroll on the screen. Maybe we need to keep pressing the `up` button or the information sequentially displays on its own. 

If you get stuck, these lines of code such as these may be useful. (Also, don't forget to include `time.h` at the top of your code.)

{% highlight c++ %}
time_t seconds = time(NULL);
char buffer[32];
strftime(buffer, 32, "%e\n", localtime(&seconds));
sprintf((char *)stemp2, buffer);
lcd.DisplayString(stemp2);  {% endhighlight %}


[epoch]: http://www.epochconverter.com/
[time]: http://www.cplusplus.com/reference/ctime/strftime/

### feeling DOWN 
Friends are also meant to cheer us up and make us feel better when we are feeling `down`. Maybe our disco mate can help us when we are feeling sad or unmotivated. 

I want my disco mate to ask me `How are you feeling?` and then scroll through three options `R: unmotivated`, `L: sad`, `U: fine`. If I precede to press the `right` button, my disco mate will provide me with a motivational quote. If I press the `left` button, my disco mate will try to cheer me up with a funny joke. If I press the `up` button, my disco mate will respond with the message `Glad to hear`. 

You should now be able to make the scrolling message and options appear on the LCD screen by modifying the `left_pressed` function. If you feel confident with the other specifications, have a go at coding yourself before reading any further. 

It will be useful to define some new functions associated with the physical movement of the joystick. We propose these three functions. 

{% highlight c++ %}void down_right_pressed () {
}
void down_left_pressed () {
}
void down_up_pressed () {
} {% endhighlight c++ %}

Inside the scope of the original `down_pressed` function, we will call on these as shown below. If you are confused with what is going on here, go back and look at the `InterruptIn` class documentation again. 

{% highlight c++ %}right.rise(&down_right_pressed);
left.rise(&down_left_pressed);
up.rise(&down_up_pressed);
{% endhighlight c++ %}

Recall that we want the screen to bring up a motivational quote on the LCD display if the `right` button is pressed.  One idea is to generate a random number that chooses a quote from a set list. Have a go at coding this method yourself. 

If you get stuck, take a look at some of this sample code. 

{% highlight c++ %}int randNum = rand()%(max-min + 1) + min;
   if(randNum==1){
   sprintf((char *)stemp, "%d", randNum);
   lcd.DisplayString(stemp);
   lcd.Clear();
   uint8_t scroll[]= "     Your only limit is you";
   lcd.ScrollSentence(scroll, 1, 200);
   lcd.Clear();
   }{% endhighlight c++ %}

Now, try and repeat this process for the `left` and `up` buttons. 

### LEFT field 

Maybe our disco mate can also do something a bit more `left` field. What if we could make a small game to play when the `left` button is pressed? Take a few minutes to brainstorm some ideas! 

Perhaps you could code a mental arithmetic game? There are loads of possible ways to implement this! 

Some ideas to get you thinking:

-The count starts at 0. The player has to add the value of digits that appear on the left side of the screen and subtract values of digits that appear on the right side of the screen. 

-The player does not know how many numbers will appear. When the numbers stop appearing the player must press the `up` button to increase the final count by 1 and the `down` button to decrease the final count by 1. When the player has reached what they believe is the final count they press the `centre` button. 

-If their count was correct the screen displays a congratulatory message. If they are incorrect, the screen says `Oops you lost count`.

-Can you think of anyways to make the game harder? Could you incorporate multiplication? 

### RIGHT one

We have looked at ideas for what the disco mate does when the `centre`, `up`,`down`, and `left` buttons are pressed. Can you come up with a `right` one? 

### Wrapping it up 

Hopefully you are all `right` and have made a disco mate that you could wrap up and give to one of your human friends. Pretty cool to have a device that is `up` to date, there for you when you are `down`, and has a bit of entertainment for any of your `left` over time. Not to mention that you were `right` in charge of any extra or personal features. If you had any interesting or innovative ideas that you would like to share, feel free to comment below! 

Lightweight, cheap devices are going to continue to increase in number and play a more significant role, particularly as we enable them with internet connectively. Since you have had some practice using `mbed` in this tutorial, you are well equipped for many future IOT related projects! 




