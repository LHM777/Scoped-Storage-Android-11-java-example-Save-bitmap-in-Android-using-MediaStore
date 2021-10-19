# What is Scoped Storage?
Android 10 put a greater emphasis on privacy and security. Android 11 continues this emphasis and gives you many tools to achieve this. One of those tools is scoped storage. This feature impacts your app in a big way if you’re leveraging local file access.

With scoped storage, an app no longer has direct access to all the files in external storage. If an app wants to manipulate a file that it didn’t create, it has to get explicit authorization from the user. These request prompts appear for each file. This provides a new level of control for the user. They can now decide what an app can or cannot do with their files.

Because this can have a potentially heavy impact on existing apps, Android 10 provides an opt-out mechanism. When enabled, it allows an app to work without any of these requirements. The caveat here is that when your app targets API 30 (Android 11), scoped storage starts to be mandatory.

So if your app uses device storage, it’s time to start preparing it for scoped storage.
