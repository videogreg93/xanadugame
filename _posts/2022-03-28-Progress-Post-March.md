---

tags: "LibGDX Calamity-Sanctuary Progress"

---

A lot of things have changed since last year. Here's an overview of all the major changes in the past couple of months

### Removal of the overworld sections

I've gone back and forth on the idea of including an isometric overworld section in between battles reminiscent of Megaman Battle network, the main inspriation for Calamity Sanctuary. At first there was none out of simplicity, then at one point as an experiment I started tinkering around with implementing isometric sections in the game, for various reasons, mainly because I wanted to see if I was capable of not only implementing everything needed to handle isometric logic, but also create interesting procedural maps. I also thought it would be a great way to differentiate myself from popular card based roguelites such as Slay the Spire and Monster Train, which include node based traversal on a map between battles.![img](..\assets\isometric-example-cs.png)

In the end, the isometric sections turned out to be too much of a headache. My collision detection code was always wonky, the procedural generation, while definetly servicable, was difficult to tweak into something interesting, and in the end I couldn't see what the isometric traversal actually brought to the game. After much debate, I ended up scrapping the idea, and despite my quest to differentiate myself my avoiding a map with nodes, that's what I ended up implementing.

### Adding a Map

<video autoplay loop src="../assets/small_map_test.webm"></video>

To put it simply, this is basically the same as the Slay the Spire map. Create a 5x20 array, iterate over it a bunch of times while linking nodes together, and finally clean up to make things more interesting. Set each node type based on probabilaty and certain rules (e.g. no shops on the first floor, no repeated rooms for certain types, etc.) I still need to make it prettier, especially the background, but I already have a couple of neat things in place, such as animated icons and links.

I'm still debating on keeping the information at the bottom of the screen. One one hand, I think be able to see your deck composition at a glance is great for deck building, but on the other the current implementation is pretty ugly and takes up a good amount of screen real estate. 

### Added a World Screen

<video autoplay loop src="../assets/world_test.webm"></video>

This is something I've wanted to add ever since I saw the amazing [Pixel Planet Generator](https://deep-fold.itch.io/pixel-planet-generator). The idea is that after each area boss, players will have a choice between different worlds, each with their own enemies, unique events, etc. This is simply another layer of decision making for players : Do they choose the more difficult planet in hopes of obtaining better cards, or do they visit the ice planet, since they picked up a couple of fire cards in the previous world? (Elemental weakness hasn't been implemented yet, but it's something I've been thinking about more and more lately)

Right now it's mostly just for show, as all planets have the same encounter and loot tables, but everything is in place to make them eventually unique. Once I have a bigger enemy pool and when I have a better idea of how difficult each of the enemies are I'll be better equipped to separate them into different tables based on difficulty.

### Removed a lot of copyrighted material

Given that Calamity Sanctuary is inspired by Battle Network and the fact that I have no artistic capabilities, it was only natural for me to borrow copyrighted sprites when this game was barely a project. However, I'm now well past that point, and if I want to be able to show off more of the game, I really couldn't continue having a lot of the major sprites be copyrighted material. While there are still some copyrighted assets remaining, I feel like I've removed a good amount of the most prominent ones, such as the main character, enemies, and a bunch of the card art. For the rest, I try to keep a list updated of assets that need to be replaced, so I can go though it and remove items one by one.

I've also given myself as a rule to avoid adding any more copyrighted material. Like tech debt, replacing the assets just ends up costing more time in the end, and I wouldn't want to include any assets I don't own inadvertently into the final product.

### New Keyword - Combo

A new card keyword, "Combo" was added. When a card with Combo is played, it will add a new card to your hand. For example, the dagger card, when played, will not only swipe at the square in front of you, but will also add a "Throwing Dagger" card to your hand, which can deal damage across the screen. Combo cards incentivise build which want to play a lot of cards. There are also Combo specific cards and daemons, like the Specialist card, which increases the damage of cards created via Combo, and Resourceful, which will let you draw a card when you play a combo card.

The Combo keyword went through a couple of iterations before landing on it's current implementation. The initial idea was that playing a Combo card would transform other identical cards in your hand into more powerful versions. There were however 2 major issues with this version :

- There were a bunch of technical and gameplay side effects to consider and I didn't have a good answer for any of them. Would transformed cards revert to their original form when discarded? Do transformed versions of cards also transform "original" versions of cards when played? Was transforming a card considered discarding a drawing a card?
- I felt that this version incentivised too much just trying to obtain copies of the same card. Since Combo cards wouldn't be very useful without at least one or two other copies in your hand, it was in player's best interest to simply avoid picking other cards and going "all in" on one card in particular. I don't think these would lead to very interesting situations, which is why I abandoned the idea.

### New Enemies, Cards, and more

A lot more content was added in these past months. Here are a couple new enemy sprites and animations

![buzz-sheet](..\assets\buzz-sheet.gif)![idle](..\assets\snakey)![idle](..\assets\crow_idlew)

### Conclusion

A lot has been done since the beginning of the year, but like always, I feel like I very far from an actual finished product. Hell, I'm probably still pretty far from an acceptable Alpha build of the game. However, for those of you who are curious, you can always find the latest build on my [Itch.io page](https://videogreg93.itch.io/calamity-sanctuary). Thanks for reading.