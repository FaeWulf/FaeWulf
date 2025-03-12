Create: "2025-03-11T10:40:59.954Z"
Edit: "2025-03-11T10:40:59.954Z"
Tags: "minecraft mixin tutorial"

> [!NOTE]
> **TL;DR**  
>Some Mixin injectors (`@Redirect`, `@ModifyConstant`, `@Overwrite`) can cause **hard conflicts** when multiple mods modify the same method/class, leading to crashes. 🚨
>
> **Safer alternatives using MixinExtras**:
>
> - Use `@WrapOperation` instead of `@Redirect` 
> - Use `@ModifyExpressionValue` instead of `@ModifyConstant` 
> - Avoid `@Overwrite`; try other injectors instead 
>
> **Other conflicts**:
> 
> - Implementing interfaces in Mixin classes can cause crashes.

---

If you are learning or making a Minecraft mod and using Mixins, beware that some types of injectors can cause conflicts if two mods modify the same method or class. This can lead to crashes and compatibility issues between mods.

## 📃 Example of a Conflict

For example: I made a mod that removes the anvil experience limit. To achieve this, I wrote the following Mixin to change the limit value:

```java
@Mixin(AnvilMenu.class)
public class AnvilBlockMixin {
    @ModifyConstant(method = "createResult", constant = @Constant(intValue = 40, ordinal = 2))
    private int ModifyLevelLimit(int original) {
        if (!ModConfigs.no_level_limit_anvil)
            return original;
        
        // Maybe this is enough? 🤔
        return Integer.MAX_VALUE;
    }
}
```

This works perfectly!!—until you install another mod that modifies the same thing. Then, 💥 BOOM! The game crashes. 🛑

This is just one example of how compatibility issues can occur when multiple mods inject into the same place. Worse still, if another modder does something similar, the conflicts multiply.

## 🛡️ How to Avoid Mixin Conflicts

This post is intended to help new modders avoid these conflicts when using mixins. And show you how to achieve the same results without any losts.

### 📌 Prerequisites

To follow these recommendations, install MixinExtras. 
If you don’t know how, check out this tutorial: [MixinExtras Setup](https://github.com/LlamaLad7/MixinExtras#Setup). 
There is also a useful [wiki](https://github.com/LlamaLad7/MixinExtras/wiki) for more details.

### Injectors to Avoid and Their Alternatives

  
Alright lets begin! As far as I know, there are 3 type of injectors you should avoid using:

####  `@Redirect`

Instead of `@Redirect`, use `@WrapOperation` from MixinExtras. The difference is that `@WrapOperation` allows multiple mods to chain their modifications instead of overwriting each other.

####  `@ModifyConstant`

Instead of `@ModifyConstant`, use `@ModifyExpressionValue`. The syntax is slightly different. Here's how you can replace the original example:

```java
@Mixin(AnvilMenu.class)
public class AnvilBlockMixin {
    @ModifyExpressionValue(method = "createResult", at = @At(value = "CONSTANT", args = "intValue=40", ordinal = 2))
    private int ModifyLevelLimit(int original) {
        if (!ModConfigs.no_level_limit_anvil)
            return original;
        
        return Integer.MAX_VALUE;
    }
}
```

####  `@Overwrite`

`@Overwrite` should be avoided as much as possible. There is no solid workaround, but you should always try to use alternative injectors instead of completely replacing a method.

### ⚡ Other Causes of Conflicts I Want To Mention

Besides these injectors mentioned above, another common cause of crashes is using the same interface in multiple Mixin classes.

For example, if you want to make `small flowers bonemealable`, you might try something like this using built-in Minecraft interface:

```java
@Mixin(FlowerBlock.class)
public class FlowerBonemealMixin implements BonemealableBlock {
    @Override
    public boolean isValidBonemealTarget(...) {
        return true;
    }

    @Override
    public void performBonemeal(...) {
        // Apply bonemeal effect 
    }
}
```

However, if another mod also mixins in `BonemealableBlock` to `FlowerBlock`, this can lead to conflicts.
So, avoid implementing new interfaces in Mixin classes.
A workaround that I found works is to copy the whole interface into a custom interface and use custom interface instead. Well, it works! Until someone decides to use your interface 🤷‍♂️

## The end

Hope this small post can help you create stable and more compatibility mods!
