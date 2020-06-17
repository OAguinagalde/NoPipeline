# This fork

It's ~~exactly the same as the original repository, only now it's~~ a .Net Core application instead.

* (17/06/2020) Now it also fixes an issue regarding the usage of full paths instead of relative paths.

----

# NoPipeline

![NoPipeline](/pics/NoPipeline.png)

You know, Monogame is nice. C# cross-platform game framework which is a pretty good base for an engine or a game.

What certainly is not nice - their Pipeline Tool. 
For some reason, the only Monogame resource manager is a total pain in the ass to work with: clunky external GUI, no ability to add whole directories. And of course, bugs. Countless, countless bugs.


## So what shall we do about all this?

Fear not - NoPipeline comes to the rescue. It's an addon for Pipeline Tool, which generates and updates `.mgcb` config for you. You can safely add, delete and move around resource files right in Explorer - NoPipeline will do the rest for you.

Additionally, you can make resource files watch other files! Let's say, you got Tiled map project. It has one main `.tmx` file and a bunch of textures and tileset files. But Pipeline Tool has referenced only `.tmx` file, so if you
update only texture or only tileset, you have to either update the `.tmx` or do a manual rebuild, because Pipeline Tool doesn't know about files other than `.tmx`. 


With NoPipeline you don't have to do any of that - just set `.tmx` file to watch textures and tilesets - and Pipeline Tool will detect and update everything by itself.


## Sounds cool and all, but I don't want to migrate to some other tool.

You don't have to. NoPipeline is not a Pipeline Tool replacement - it's an addon. You can add or remove NoPipeline at any point in development, and nothing will break. NoPipeline won't override resources which already exist in the `.mgcb` config and will leave a perfectly valid config after itself.


## Now we're talking. How do I integrate this thing in my project?

First of all, install the [latest release](https://github.com/Martenfur/NoPipeline/releases/latest) of NoPipeline. After that, you will need a NPL config. NPL config is what NoPipeline uses to generate MGCB config. Inside it looks like this:


```json
{
	"content": 
	{
		"textures": 
		{
			"path": "Textures/*.png",
			"recursive": "True",
			"action": "build",
			"importer": "TextureImporter",
			"processor": "TextureProcessor",
			"processorParam": 
			{
				"ColorKeyColor": "255,0,255,255",
				"ColorKeyEnabled": "True",
				"GenerateMipmaps": "False",
				"PremultiplyAlpha": "True",
				"ResizeToPowerOfTwo": "False",
				"MakeSquare": "False",
				"TextureFormat": "Color",
			}
		},
		"specificFile": 
		{
			"path": "Path/To/File/specificFile.txt",
			"recursive": "False",
			"action": "copy",
		}
	}
}
```
NPL config is essentially a JSON. Config above has two file groups: `textures` 
and `specificFile`. Each file group describes one specific resource type. 
File groups can contain whole directories or single files.


Let's look at an each parameter:
- `path` is a path to the resource files relative to the main Content folder. 
Here are some examples:
	- `Graphics/Textures/texture.png` will grab only `texture.png` file.
	- `Graphics/Textures/*.png` will grab any `.png` file.
	- `Graphics/Textures/*` will grab any file in the `Textures` directory.
- `recursive` tells NoPipeline to include resource files from subdirectories.
For example, if set to `True`, and the `path` is `Graphics/Textures/*.png`,
files from `Graphics/Textures/Subdir/` will be grabbed as well. If set to 
`False`, they will be ignored.
- `action` tells what action has to be done for this file group. Can be `build`
or `copy`.
- `importer` tells what importer should be used for building.
- `processor` tells what processor should be used for building.
- `processorParam` is an optional list of processor parameters, if resource 
has any.


There is also an optional `watch` parameter. Its usage looks like this:

```json
{
  "content": 
  {
    "spriteGroup": 
    {
      "path": "Graphics/*.spritegroup",
      "recursive": "True",
      "action": "build",
      "importer": "SpriteGroupImporter",
      "processor": "SpriteGroupProcessor",
      "watch": 
      [
        "Default/*.png",
        "Default/*.json",
      ]
    },
  }
}
```
With `watch` parameter present, all the `.spritegoup` files will be built by Pipeline Tool, if any `.png` or `.json` file will be changed. Note that all the paths listed in `watch` are relative to the main `path`, so final paths  will look like this: `Graphics/Default/*.png`.

But that's not all. NoPipeline also provides an extended reference management. Add `references` section into your `.npl` config like this:

```json
{
	"references":
	[
		"%PROGRAMFILES%/YourLibDir/Library.dll",
		"RelativePath/RelativeLibrary.dll",
	],
	"content": 
	{

	}
}
```
With NoPipeline you can use environment variables like `%PROGRAMFILES%` - something Pipeline Tool can't do by itself. If referenced libraries are missing, NoPipeline will delete their entries from config. Additionally, add references the old way from the Pipeline Tool - NoPipeline will not delete them unless the files themselves doesn't exist.

You can also take a look at the sample config `SampleContent.npl`, which comes included with the program.


With NPL config done, save it in the same directory as MGCB config and name it the same as it. So, if your MGCB config is named `Content/Content.mgcb`, your NPL config should be `Content/Content.npl`

You can also include NPL in Visual Studio project, if you want.


After all that, open `.csproj` file in text editor and find these entries:

```xml
<Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
<Import Project="$(MSBuildExtensionsPath)\MonoGame\v3.0\MonoGame.Content.Builder.targets" />  
```

Insert `<Import Project="$(MSBuildExtensionsPath)\NoPipeline\NoPipeline.targets" Condition="Exists('$(MSBuildExtensionsPath)\NoPipeline\NoPipeline.targets')"/>`
**before** content builder entry, so it would look like this:

```xml
<Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
<Import Project="$(MSBuildExtensionsPath)\NoPipeline\NoPipeline.targets" Condition="Exists('$(MSBuildExtensionsPath)\NoPipeline\NoPipeline.targets')"/>
<Import Project="$(MSBuildExtensionsPath)\MonoGame\v3.0\MonoGame.Content.Builder.targets" />  
```

And from this point you can start forgetting about Pipeline Tool. : - )

If you want more seamless pipeline-forgetting experience, you can check out [Monofoxe Engine](https://bitbucket.org/Martenfur/monofoxe/src), with NoPipeline integrated out of the box.

## All the other stuffs. 

The thing is licensed under MPL 2.0, so you can use it and its code in anything you want for free.


Huge thanks to [MirrorOfSun](https://github.com/MirrorOfSUn), who wrote most of the code.


*Don't forget to pet your foxes.*
