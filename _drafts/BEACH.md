# Beach

Spending one week at the beach and playing Popcap games on the flight there does things to you. Like make you want to start a new project: A *small*, puzzle/arcade like game. Hence, **BEACH** was born.

What do you pick up at the beach? Rocks, starfish, that kind of think. However, I didn't have anything like that in my current asset library, so I decided to go with the most *beachlike* thing I had : Fruit.

![fruit_sheet](..\assets\fruit_sheet.png)

I had been playing [Bejeweled 2](https://en.wikipedia.org/wiki/Bejeweled_2) on the plane, so I didn't want "swapping two gems to make a line" to be the main interaction. The only other game I have played in the genre is [Hexic](https://en.wikipedia.org/wiki/Hexic), so I didn't want to be rotating gems neither. So I came up with the following :

- You can slide complete rows or columns of *gems* (fruit from now on).
- You capture fruit when you make a rectangle of fruit of the same type, including the fruit of other types that end up inside that rectangle.

After a couple hours I had something like this

<video src="..\assets\beach.webm"></video>

> [!NOTE]
>
> The above is actually after more than a couple of hours, the original version did not have that background nor the almost imperceivable rotation of fruit, but this is the earliest video I could find.

##  What's up with that background?

This being a beach game, I was tired of looking at a static background and wanted something more... Frutiger Aero. So I found one image I really liked from [this reddit post](https://www.reddit.com/r/FrutigerAero/comments/14qmlni/over_50_frutiger_aero_wallpapers_2008ify_your_pc/) as a placeholder.

![fruitiger_background_1](../assets/fruitiger_background_1.jpg)

However cool the image was, the interesting part of the image was also smack dab where the action happens, making it visually hard to read what is going on. I started playing around with a sine wave distortion shader ([example here](https://www.shadertoy.com/view/MsX3DN)), and after pushing the values a bit high, ended up with what you see up top.

## The controls seem tedious

Absolutely. My problem is that I never think about mouse controls, simply because 
