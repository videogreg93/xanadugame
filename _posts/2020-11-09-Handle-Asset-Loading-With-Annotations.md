---

tags: "LibGDX LibKTX"
comments: true

---

The **AssetManager** handles loading and unloading various assets such as *Textures* and *Sounds*. Rather than loading them on use, you can use the Asset Manager to load all necessary assets at game launch, or at the beginning of a level. It also serves as a central point for all assets, making it easier to avoid reimporting the same asset multiple times.

```kotlin
val manager = AssetManager()
manager.load("data/mytexture.png", Texture::class.java)
/*
 * Load all assets that we called 'load' on.
 * As long as update returns false, we havn't finished loading
 */
while (!manager.update()) {
  // do nothing
}
// Use the asset
val texture: Texture = manager.get("data/mytexture.png", Texture::class.java);
```

While using the Asset Manager is much better than loading them on the fly, managing the `load` and `get` calls can very quickly start to get messy, not to mention all of the redundant code. Ideally, we would like to declare an asset once, and have the engine handle the loading and whatnot for us. 

# Proposed solution

```kotlin
@Asset
val textureAsset = AssetDescriptor("data/myTexture.png", Texture::class.java)

val texture: Texture = texture.get()
```

We've introduced the `@Asset` annotation, which will let us tell our engine that the following field is an asset to be loaded by our asset manager. Here is what the annotation declaration looks like.

```kotlin
@Target(AnnotationTarget.FIELD)
annotation class Asset()
```

By using this simple annotation we've done two things. We avoid having to drag along our asset manager wherever we want to declare assets, and we've created one single reference to handle both telling our asset manager to load the object and also serving as a reference point to obtain the actual asset.

However, simply adding the annotation won't put everything into motion. Let's see how we actually load the assets at startup.

# Loading the Assets

Now that we've annotated our fields, we need to fetch this information and queue up the loading. Let's take a look.

```kotlin
private fun loadAllAssets() {
  val reflection = MainContext.inject<Reflections>()
  val assets = reflection
  .getFieldsAnnotatedWith(Asset::class.java)
  .mapNotNull {
    it.isAccessible = true // Workaround for having this work despite wanting to limit scope of certain assets
    it.get(null) as? AssetDescriptor<*>
  }
  
  assets.forEach {
    assetManager.load(it)
  }
}
```

This basically replaces all of our `load` calls from the initial method. We're using the `org.reflections:reflections` library to fetch all fields with the `@Asset` annotation as `AssetDescriptors` and loading them all. Although I haven't talked about it so far, `AssetDescription` is an existing LibGDX class which basically holds the path and filetype information of an asset.

Once we've queued up the assets, we need to start loading them all. This hasn't changed from before so we simply call `assetManager.update()` each frame, which returns `true` if we're finished loading everything.

# Obtaining the Assets

The final piece of the puzzle is the `get` extension function, which lets us obtain the actual asset via the same reference with annotated earlier.

```kotlin
fun <T> AssetDescriptor<T>.get(): T {
    return assetManager.get(this)
}
```

Using generics lets us omit the actual class name when permitted, which is useful in keeping things short and concise. 

# Putting it all together

Let's look at how it all comes together. For reuse in all my projects, I've wrapped everything found here in my own `AssetManager` class. Here is what it looks like (certain lines omited or changed for brevity and completeness).

```kotlin
// package ...
import com.badlogic.gdx.assets.AssetDescriptor
import ktx.freetype.registerFreeTypeFontLoaders
import org.reflections.Reflections
import com.badlogic.gdx.assets.AssetManager as GdxAssetManager

class AssetManager {

    init {
        assetManager.registerFreeTypeFontLoaders()
        loadAllAssets()
    }

    fun install() {
        assetManager.registerFreeTypeFontLoaders()
        loadAllAssets()
    }

    fun update(): Boolean {
        return assetManager.update()
    }

    fun progress(): Float {
        return assetManager.progress
    }

    private fun loadAllAssets() {
        val reflection = Reflections("com.xanadu.section", FieldAnnotationsScanner(), TypeAnnotationsScanner(), SubTypesScanner())
        val assets = reflection
                .getFieldsAnnotatedWith(Asset::class.java)
                .mapNotNull {
                    it.isAccessible = true
                    it.get(null) as? AssetDescriptor<*>
                }
        assets.forEach {
            assetManager.load(it)
        }
    }

    companion object {
        private val assetManager: GdxAssetManager = GdxAssetManager()
        
        fun <T> AssetDescriptor<T>.get(): T {
            return assetManager.get(this)
        }
    }
}

@Target(AnnotationTarget.FIELD)
annotation class Asset()
```

# Improvements for the Future

- Right now we're using reflection at runtime to obtain all the assets. Ideally we should be generating this code with something like **Kotlin Poet** at compile time, as bigger projects will take some time finding all of our annotated fields.
- Our `@Asset` annotation is very simple, we could enrich it with some helpful functionalities. For example, we could add a `Level` enum to denote which levels of our game uses this asset, so instead of loading all assets at the beginning, we could load necessary assets at the beginning of each level, to avoid a large load time at startup.

