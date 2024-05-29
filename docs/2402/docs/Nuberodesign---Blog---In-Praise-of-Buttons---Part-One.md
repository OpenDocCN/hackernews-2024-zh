<!--yml
category: 未分类
date: 2024-05-27 14:30:29
-->

# Nuberodesign > Blog > In Praise of Buttons – Part One

> 来源：[https://www.nubero.ch/blog/009/](https://www.nubero.ch/blog/009/)

In Praise of Buttons – Part One

Form *is* Function in Graphical User Interfaces

By Niko Kitsakis, January 2024

In any design discipline there are always certain trends. One of these trends seems to be that buttons are now considered uncool. It doesn’t matter if they are buttons on physical objects or in graphical user interfaces.

Buttons aren’t new. And for exactly that reason, they are uncool. Like hammers and paperclips, they have been around for a long time and they work. In fact, they have been around for a long time *because* they work.

That is a problem if you’re a dishonest designer. After all, how do you tell your client that you’ve just reinvented the wheel? You can’t just use boring old buttons in your shiny new product. So what do you do? You redesign the look or the function of a button (or both) and sell it as “the newest development in design” – totally dis­re­gar­ding the needs of the user and compromising your integrity as a designer along the way.

Buttons on Screens

In graphical user interfaces, we have seen an increase in buttons recently that consist merely of text or icons, without a clear, visible button shape being present. This insipid, uninspired mediocrity, exemplified by Google’s “Material Design” or – even worse – IBM’s “Carbon Design System”, was popularised by Apple’s iOS 7 and its equally miserable “Flat Design” aesthetics. This lazy minimalism is often considered modern and streamlined, but we must ask: Is it also user-friendly?

The answer is clearly: No, it is not!

Consider the image below: In the first row we see a series of icons that are supposed to be buttons. The only way you could potentially recognise them as such, however, would be if they were implemented in a user interface. So it is in fact only the *context* that lets you recognise them as buttons, not their *appearance* by themselves.

Two rows of buttons. Which appear more inviting to interact with?

Compare the icons in the upper row with their counterparts in the lower row. Here, these same icons are embedded in button shapes. This does several things at the same time. First, the button shapes act as *signifiers*. That means that they let the user know that an action can occur there. The icons in the upper row do no such thing. They don’t commun­icate their ability to be pressed – to be *used* really – in any way.

How important signifiers are becomes clear when you look at the physical objects around you. Take the Swiss Army Knife in the picture below for example. The small groove that you see in the main blade (and in almost every hinged tool of a Swiss Army Knife) is called the “nail nick”. Not only does this nail nick allow you to insert your finger­nail and pull the tool out, it also *looks like it allows for precisely that.* Pay attention to its exact shape too. It mirrors the slight curvature of a fingernail. And it doesn’t stop there: The angles of the walls *inside* the groove are contoured in such a way that your fingernail is guided deeper into the nail nick, ensuring a secure hold for pulling.

This is the nail nick in the main blade of a Swiss Army Knife. It gives the user a clue that an action can occur here and that this action probably involves a fingernail.

This should make it clear why there is a reason that the virtual buttons in our graphical user interface should indeed look like buttons: They should commu­nicate that they can be used. When they just look like icons, they don’t do that.

Look at the image of the buttons once more, and this time, try to find groups that belong together. The icons for plus and minus and for left and right for instance form two groups while the magnifying glass stands separated. In the lower row, where the buttons have uniform shapes, it’s easier to notice the differences in spacing between them.

There is another thing to consider. Take a look at the next picture.

The buttons in the upper row not only *communicate* what they do badly, they *function* badly as well.

The pink colour shows where a button can actually be clicked or touched, that is to say, it shows the region where the software will register an input. In the upper row, that is quite a small area if you think of it in terms of square-pixels. That being said, the clickable area in mouse-interfaces *might* sometimes be the same in the upper row as in the lower row. In touch interfaces however, that is often not the case. There, the actual outline of the graphic – let’s say the minus – is often the only thing that can be touched. So when you quickly tap your screen and don’t hit the target precisely, you might completely miss the button.

Text in Buttons

The importance of button shapes becomes even more apparent, when the buttons are actually just words, not icons. The example below shows the Terms and Conditions of an iOS update. You can see that the two words, “Agree” and “Disagree”, don’t stick out too much in an environment that already has a lot of text in it. Apple’s way of showing the user that the two words are actually buttons is done solely with colour. The default option of agreeing is also not differentiating itself enough from the disagree option. The only difference is a slightly bolder (but not really bold) typeface. And lastly, speaking of type­faces, the one that Apple chose – specially designed actually – is also not the best for user interfaces.

Everything that can go wrong, will go wrong: The people who designed this blindly followed the notion, that removing everything they perceived as ornamental would result in simplicity.

Now compare the example from Apple above with my redesign below. Look at the difference that the button shapes and a better choice of typeface make. The typeface in question is called “FF Unit” and has less ambiguous letter shapes than Apple’s “San Francisco”. Granted, this is less important in long text than it is in individual words, but since we want the same typeface for both our buttons and the legal text, choosing a typeface *like* FF Unit becomes rather obvious. Lastly, I used a slight shadow to set apart the scrollable text of the Terms and Conditions from the surrounding interface. This too makes for a better visual distinction between the different elements on screen.

![The Terms and Conditions redesigned](img/8351efc82a4399cfbb083e93ebec4f95.png)

Just because a user interface uses 3D-buttons and some shading doesn’t mean that it has to look tacky. In fact, if you have to make the choice between tacky-but-usable and minimalistic-but-hard-to-use, tacky is the way to go. You don’t have to make that choice though: It’s perfectly possible to create something that is both good-looking *and* easy to use.

Direct Manipulation

The physical world that we evolved in is one where every action has an effect. When you push the coffee mug on your desk and it moves away from you, it will make a sound. You might also be able to see little waves forming in your coffee as the small vibrations from the mug sliding on the desk transmit to the liquid. All of this feedback is expected by our brain to come to us through our different senses.

Think about what this means: You will get haptic feedback through your fingertip, giving you information about weight, temperature and texture. There will be acoustic feedback from the ceramic sliding on your desk. And you will get visual feedback from seeing your finger and the mug moving, and the little waves on the surface of the coffee. Since our brain builds a model of the world and then compares reality to it, these different feedbacks are both *expected* and *interconnected*. It would be extremely weird, for example, if the mug did not make any noise at all while it slides across the table or if the liquid wouldn’t move inside the mug.

A button in a graphical user interface that has no button shape will likely give you no feedback either. While it might actually have an alternative state that gets activated when you touch it (a change of colour for instance), you are probably going to obstruct that with your finger. This is another advantage of buttons that look like buttons. Because they usually stick out from under a fingertip, you can see the press-down animation clearly. Direct feedback from direct manipulation. See the animation below:

 <https://nuberobucketone.s3.eu-central-1.amazonaws.com/blog009/hand.mp4> 

While the user cannot be sure that his tap on the upper arrow was registered by the device (and might therefore try multiple times), the “button depressed” state of the 3D-button signals to him that his action was successful.

Why is this important? First of all, it should be self-evident that a feature of the world that our brains evolved around, namely feedback, should not be removed just to satisfy a fashionable trend. As mentioned above, we expect some sort of feedback from practically everything in the world. But there’s another thing to consider: The button you just pressed might make some webpage load or do something else which will take a second or more before its effect is apparent. While you wait for that to happen, a button that showed you that it got pressed – that gave you feedback – is going to give you reassurance that you actually tapped it correctly.

Visual feedback is, of course, just one way to solve this. Acoustic or haptic feedback also works well. Even better is a combination of these. Remember the coffee mug, whose weight and vibration you can feel, while you also see it moving and hear it sliding across the table. This sort of multi-sensory feedback is what you should be going for – if the circumstances allow it.

 <https://nuberobucketone.s3.eu-central-1.amazonaws.com/blog009/iphone_album.mp4> 

One last example: In this video, you can see the Photos app as it appears in iOS 17, alternating with a redesign done by me. Apple is using lazy button shapes for “Select” and the ellipsis character (…) on the right. On the left however, the button for going to your albums is just the word “Albums” and a little arrow shape. Why those two different concepts on the same screen? In the redesign, everything that works like a button also looks like one. This, in my opinion, is how it should be done.

Misguided Sentimentalism?

Critics of my position may point to what they believe is some sort of sentimentalism for old user interfaces on my part. It’s true that the problems I point to in this piece are of the kind that I consider solved in many of the older user interfaces. That has nothing to do with being sentimental however. Products have to work properly. If a button is the right choice, use a button. If it’s not, don’t. But if you are going to implement a design element that *works* like a button, it should *look* like one too.

Does *every* virtual button – every button in a graphical user interface – have to look like a pressable, physical 3D button? Of course not. They also don’t need to look exactly like my redesigns either. On a case-to-case basis it might even be better to do something else entirely.

The whole idea is to reduce cognitive load. And since the brain works by recognising patterns and dividing the environment up into areas, this reduction is best done by making elements with different functions appear markedly distinct from one another. It is, in other words, a fallacy to believe, that the brain has an easier time if everything looks “simplified” in the way which happens with the flat design doctrine. The opposite is the case.