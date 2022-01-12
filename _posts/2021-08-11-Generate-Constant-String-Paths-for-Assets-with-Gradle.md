---

tags: "LibGDX Gradle Gradle-Task Code-Generation"

---

Unless you've created a fleshed out engine inside of LibGdx, chances are you're manually providing asset paths for things like textures, sounds, etc. Here's a typical example of creating a `Texture`. 

```kotlin
val texture = Texture("player/overworld/player.png")
```

There are many problems with this approach, notably :

- It's easy to introduce a typo, which will probably crash at runtime when trying to fetch the asset.

- It's hard to know if the asset is already being used somewhere else. Sure, you can `ctrl+f` "player/overworld/player.png" but if you did something like this you won't find your asset :

  ```kotlin
  const val PLAYER_FOLDER = "player/"
  val texture = Texture("$PLAYER_FOLDER/overworld/player.png")
  ```

  

Since we know all of our assets at compile time, what if we could generate symbols to represent our asset paths? We could even keep the same hierarchy as our file system. Then, instead pf providing a hardcoded string, we could do something like this: 

```kotlin
object Player {
    object Overworld {
        const val player = "player/overworld/player.png"
    }
}
// [...]
val texture = Texture(Player.Overworld.player)
```

Let's use Gradle to create a task that we can run whenever we add new assets to our project that will let us generate this file.

### Creating a Gradle Task

Creating a Gradle Task is as easy as writing `task yourTaskName() {}` in your `build.gradle` file. Gradle tasks let you do many powerful things, like generate code at compile time, publish your project to a service like Itch.io, and more. Leveraging Gradle can make routine tasks completely automated.

Lets create a dummy task to make sure everything is working correctly. Navigate to your top level `build.gradle` file, and inside the `project(":core") ` section, write the following task

```groovy
task generateAssets() {
    println("Hello world!")
}
```

You should see the 'Load Gradle Changes' button appear in the top right corner (it looks something like this ![image-20210811152352131](..\assets\image-20210811152352131.png)) Hit that button to let Gradle know you've made some changes. If everything goes well, you should be able to see your new task by navigating in the Gradle tab in the top right corner, and searching for your new task (regenerateImages in the pic below). Double click that and you should see "Hello world" print in the console below.

![image-20210811152539276](..\assets\image-20210811152539276.png)

### Generating Some Code

Code generation can seem like a daunting task, but in reality it's just writing text to a file. Lets begin by declaring a couple of constants

```groovy
task generateAssets() {
    def project = '../core' // Core project folder, should probably be kept as-is
    def assetsFolder = '../core/assets' // Assets folder, should probably be kept as-is
    def source = 'src' // Source folder, should probably be kept as-is
    def pack = 'com.mypack.assets' // package name of the file to be generate. Replace with your top level package name
    def fileName = 'Assets.kt' // File name, feel free to rename as needed.
    
    def newLine = System.getProperty("line.separator") // OS-agnostic new line, will be used in code generation
    def builder = new StringBuilder() // This will be responsible for building up the 'Assets.kt' file
}
```

So far we've just defined a couple of things related to our project. Even if we end up only using these once, I like to declare them all at the beginning, so it's easier if I end up wanting to move things around, or if I want to copy this for another similar task, just with different parameters. 

Lets start generating out Kotlin code. We want to end up with something like this:

```kotlin
package com.mypack.assets

// Generated file, do not modify

object Assets {
    
}
```

We could use a specialized library for code generation such as [Kotlin Poet](https://square.github.io/kotlinpoet/) but for a simple case like this one, it's a good idea to just write it out ourselves.

```groovy
task generateAssets() {
    def project = '../core' // Core project folder, should probably be kept as-is
    def assetsFolder = '../core/assets' // Assets folder, should probably be kept as-is
    def source = 'src' // Source folder, should probably be kept as-is
    def pack = 'com.mypack.assets' // package name of the file to be generate. Replace with your top level package name
    def fileName = 'Assets.kt' // File name, feel free to rename as needed.
    def soundExtensions = ['.mp3', '.wav'] // Accepted sound types, feel free to modify
    def imageExtensions = ['.png', '.jpg', '.jpeg']  // Accepted image types, feel free to modify.
    def invalidChars = ['-', '.', ' ', ',']    
    def newLine = System.getProperty("line.separator") // OS-agnostic new line, will be used in code generation
    def builder = new StringBuilder() // This will be responsible for building up the 'Assets.kt' file
    
    builder.append("""package ${pack}

/** Generated from the assets folder. */
object Assets { 

""")
    // TODO asset generation
    builder.append(newLine).append("}")
}
```

All there's left to do to actually see this file is writing the contents of `builder` to an actual file.

```groovy
task generateAssets() {
    def project = '../core' // Core project folder, should probably be kept as-is
    def assetsFolder = '../core/assets' // Assets folder, should probably be kept as-is
    def source = 'src' // Source folder, should probably be kept as-is
    def pack = 'com.mypack.assets' // package name of the file to be generate. Replace with your top level package name
    def fileName = 'Assets.kt' // File name, feel free to rename as needed.
    def soundExtensions = ['.mp3', '.wav'] // Accepted sound types, feel free to modify
    def imageExtensions = ['.png', '.jpg', '.jpeg']  // Accepted image types, feel free to modify.
    def invalidChars = ['-', '.', ' ', ',']
    def newLine = System.getProperty("line.separator") // OS-agnostic new line, will be used in code generation
    def builder = new StringBuilder() // This will be responsible for building up the 'Assets.kt' file
    
    builder.append("""package ${pack}

/** Generated from the assets folder. */
object Assets { 

""")
    // TODO asset generation
    builder.append(newLine).append("}")
    // Write to file
    def path = project + File.separator + source + File.separator + pack.replace(".", File.separator) +
        File.separator + fileName // ../code/src/com/mypack/assets/Assets.kt
    println("Saving Assets at ${path}...")
    def assetsFile = file(path) // Point to the file
    delete assetsFile // delete the existing file if there is one.
    assetsFile.getParentFile().mkdirs() // Make the necessary directories if they haven't been created yet.
    assetsFile.createNewFile() // Create a new file
    assetsFile << builder << newLine // write the contents of 'builder' to the new file, ending with a newLine character.
}
```

Hit the 'Load Gradle Changes' button, and rerun your task. If everything is correct, you should see your new `Assets.kt` generated file.

### Walking Down the File Structure

We're able to generate a Kotlin source file with an empty `Assets` object, now we need to add our asset paths to it. Before doing that, lets write a couple of useful local functions to make things clearer.

```groovy
// previous lines ommited for clarity
ext.capitalize = { String s ->
    return s.substring(0, 1).toUpperCase() + s.substring(1)
}
ext.isImage = { String name ->
    imageExtensions.any { name.contains(it) }
}
ext.isSound = { String name ->
    return soundExtensions.any { name.contains(it) }
}
ext.isValidAsset = { String name ->
    return isImage(name) || isSound(name)
}
ext.cleanImage = { String name ->
    def newName = name
    (imageExtensions + soundExtensions).forEach {
        newName = newName.replace(it, "")
    }
    invalidChars.forEach {
        newName = newName.replace(it, "_")
    }
    // We cannot start a symbol with a number, so we add 'n' to the begining.
    if (Character.isDigit(newName.charAt(0))) {
        newName = "n$newName"
    }
    // We append _sound to a sound, otherwise we would generate duplicate symbols if an image and sound shared the same filename
    if (isSound(name)) {
        newName = newName + "_sound"
    }
    return newName
}
```

Finally, we're going to need to write the function that actually generates the strings that will contain the paths to our assets. We're going to write a function called `printFiles(files: File[])` whose goal is to take a list of files (Note: A file can mean an actual file or a directory) and generate the corresponding code. Here's the gist of it : The function iterates on each file:

- If it's an actual file, and it's a valid asset, add a new string to the string builder.
- If it's a directory, append the beginning of a new `object` to the string builder, call `printFiles` recursively with the files in this directory, and finally append a `}` to the string builder to complete the `object` definition. 

Here's what that looks like :

```groovy
// previous lines ommited for clarity
ext.printFile = { File[] files ->
    files.each {
        if (it.file && isValidAsset(it.name)) {
            // This is an asset, create a new string constant
            def varName = cleanImage(it.name)
            def varPath = it.canonicalPath.split("assets")[1].replace('\\', '/').substring(1)
            builder.append(newLine).append("const val $varName = \"").append(varPath).append('"')
        } else if (it.isDirectory()) {
            // Create a new object, and call this function recursively
            builder.append("""
    object ${cleanImage(capitalize(it.name))} {""")
            printFile(it.listFiles())
            builder.append("}")
        }
    }
}
```

All that's left now is to call this new function where we previously wrote `// TODO asset generation` 2 code blocks ago, and voilà, we're done writing our Gradle task! Here's what it looks like put all together : 

```groovy
task generateAssets() {
    def project = '../core' // Core project folder, should probably be kept as-is
    def assetsFolder = '../core/assets' // Assets folder, should probably be kept as-is
    def source = 'src' // Source folder, should probably be kept as-is
    def pack = 'com.mypack.assets' // package name of the file to be generate. Replace with your top level package name
    def fileName = 'Assets.kt' // File name, feel free to rename as needed.
    def soundExtensions = ['.mp3', '.wav'] // Accepted sound types, feel free to modify
    def imageExtensions = ['.png', '.jpg', '.jpeg']  // Accepted image types, feel free to modify.
    def invalidChars = ['-', '.', ' ', ',']
    def newLine = System.getProperty("line.separator") // OS-agnostic new line, will be used in code generation
    def builder = new StringBuilder() // This will be responsible for building up the 'Assets.kt' file
    
    builder.append("""package ${pack}

    /** Generated from the assets folder. */
    object Assets { 

    """)
    ext.capitalize = { String s ->
        return s.substring(0, 1).toUpperCase() + s.substring(1)
    }
    ext.isImage = { String name ->
        imageExtensions.any { name.contains(it) }
    }
    ext.isSound = { String name ->
        return soundExtensions.any { name.contains(it) }
    }
    ext.isValidAsset = { String name ->
        return isImage(name) || isSound(name)
    }
    ext.cleanImage = { String name ->
        def newName = name
        (imageExtensions + soundExtensions).forEach {
            newName = newName.replace(it, "")
        }
        invalidChars.forEach {
            newName = newName.replace(it, "_")
        }
        if (Character.isDigit(newName.charAt(0))) {
            newName = "n$newName"
        }
        if (isSound(name)) {
            newName = newName + "_sound"
        }
        return newName
    }
    ext.printFile = { File[] files ->
        files.each {
            if (it.file && isValidAsset(it.name)) {
                // This is an asset, create a new string constant
                def varName = cleanImage(it.name)
                def varPath = it.canonicalPath.split("assets")[1].replace('\\', '/').substring(1)
                builder.append(newLine).append("const val $varName = \"").append(varPath).append('"')
            } else if (it.isDirectory()) {
                // Create a new object, and call this function recursively
                builder.append("""
        object ${cleanImage(capitalize(it.name))} {""")
                printFile(it.listFiles())
                builder.append("}")
            }
        }
    }
    printFile(file(assetsFolder).listFiles())
    builder.append(newLine).append("}")
    // Write to file
    def path = project + File.separator + source + File.separator + pack.replace(".", File.separator) +
        File.separator + fileName // ../code/src/com/mypack/assets/Assets.kt
    println("Saving Assets at ${path}...")
    def assetsFile = file(path) // Point to the file
    delete assetsFile // delete the existing file if there is one.
    assetsFile.getParentFile().mkdirs() // Make the necessary directories if they haven't been created yet.
    assetsFile.createNewFile() // Create a new file
    assetsFile << builder << newLine // write the contents of 'builder' to the new file, ending with a newLine character.
}
```

You'll notice that the indentation and general formatting in the generated code is a bit off. Personally I don't think it matters since you'll never really be looking directly at the code, but you could easily modify the previous code block to take indentation into account. Nonetheless, here's what your generated code should look like (after applying auto formatting in IntelliJ) :

```kotlin
object Assets {
    object Battle {
        const val background = "battle/background.png"
        const val blue_bottom = "battle/blue_bottom.png"
        const val red_bottom = "battle/red_bottom.png"
        const val red_cracked_tile = "battle/red_cracked_tile.png"
        const val red_empty_tile = "battle/red_empty_tile.png"
        const val red_tile = "battle/red_tile.png"

        object Results {
            const val background = "battle/results/background.png"
            const val background_big = "battle/results/background_big.png"
            const val failure = "battle/results/failure.png"
            const val success = "battle/results/success.png"
        }
}
```

You should now be able to use these paths anywhere in your project that uses assets.