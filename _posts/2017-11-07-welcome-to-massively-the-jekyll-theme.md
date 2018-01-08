---
title: Automating Unity3D Assets Import
date: 2017-11-07 00:00:00 Z
tags:
- Unity3D,
- gamedev
description: 
<<<<<<< HEAD:_posts/2017-10-31-welcome-to-massively-the-jekyll-theme.md
tags: Unity3D, gamedev
date: 2017-11-07
image: https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/10/1443735529Fotolia_91388525_Subscription_Monthly_M-1024x768.jpg
=======
cover_image: https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/10/1443735529Fotolia_91388525_Subscription_Monthly_M-1024x768.jpg
>>>>>>> aad78505d4b2ccd854a986d0a5cdfa79e27c2066:_posts/2017-11-07-welcome-to-massively-the-jekyll-theme.md
---

Games aren't always made by a one-man-team but involve several developers, who are people akin to making mistakes,that in turn increase dev cost time. 
Importing assets is no joke, with the default settings and different rules for different parts of your project, let's start with this premise and ask ourselves: "can we write tools to prevent common, costly mistakes ?"

the answer is yes and let's discover one cool Unity3D Class.


**AssetPostprocessor:**

the AssetPostProcessor class receive callbacks on importing something. it implements OnPreProcess* and OnPostProcess* methods and apply your rules to the assetImporter instance.
therefore we can write rules that prevent some common mistakes, let's take textures for instance, we can ensure:
- setting the texture type (deault, sprite, etc..)
- making sure Read/Write is disabled.
- disabling mipmaps for anything that is 2D or UI (it is enabled by default).
- forcing a max texture size

```C#
public class AssetImportSample : AssetPostprocessor {

    void OnPreprocessTexture()
    {
        if (assetPath.Contains("_UI") || assetPath.Contains("_Sprite"))
        {
            TextureImporter textureImporter = (TextureImporter)assetImporter;
            textureImporter.textureType = TextureImporterType.Sprite;
            textureImporter.spriteImportMode = SpriteImportMode.Multiple;
            textureImporter.mipmapEnabled = false; // we don't need mipmaps for 2D/UI Atlases

            if (textureImporter.isReadable)
            {
                textureImporter.isReadable = false; // make sure Read/Write is disabled
            }
            textureImporter.maxTextureSize = 1024; // force a max texture size
            Debug.Log("UI/Sprite Audit Complete");
        }
    }
}

```
Now with the above script, is it easier for the team to comply to naming conventions rather than cherrypick settings for different parts of a project.

we can also implement the same OnPreprocessModel() method for meshes to audit some common errors:

• Make sure Read/Write is disabled
• Disabling rig on non-character models (it fixes the auto added Animator component)
• Copy avatars for characters with shared rigs
• Enable mesh compression

and of course, also applies for Audio.