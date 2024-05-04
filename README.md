<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Introduction](#introduction)
- [Getting Started](#getting-started)
   * [Installation](#installation)
   * [Initial Setup](#initial-setup)
- [Feature Overview](#feature-overview)
   * [Multi-Scope Preference Management](#multi-scope-preference-management)
      + [Player Scope](#player-scope)
      + [ProjectRuntime Scope](#projectruntime-scope)
      + [ProjectDevelopment Scope](#projectdevelopment-scope)
      + [Editor Scope](#editor-scope)
      + [Standalone Scope](#standalone-scope)
   * [Encryption](#encryption)
   * [Powerful Editor UI](#powerful-editor-ui)
   * [Preferences Item Framework](#preferences-item-framework)
   * [Updating PlayerPrefs Values of Standalone Build in Realtime](#updating-playerprefs-values-of-standalone-build-in-realtime)
      + [Real-Time Monitoring and Updates](#real-time-monitoring-and-updates)
      + [Using the `Watch` Method](#using-the-watch-method)
      + [Integration and Control](#integration-and-control)
- [Code Documentation](#code-documentation)
- [Code Concepts](#code-concepts)
   * [Prefs Class](#prefs-class)
   * [IPrefs Interface](#iprefs-interface)
   * [PlayerPrefsChangeMonitor](#playerprefschangemonitor)
- [Example Usage](#example-usage)
   * [Using Preference Items](#using-preference-items)
   * [Using Direct Code](#using-direct-code)
- [Samples](#samples)
   * [Encrypted Money Value](#encrypted-money-value)
      + [Real-Time Modifications](#real-time-modifications)
   * [Encrypted Money Value Item](#encrypted-money-value-item)
- [Preferences Editor](#preferences-editor)
   * [Toolbar](#toolbar)
   * [Preferences List](#preferences-list)
   * [Adding New Entries](#adding-new-entries)
- [Troubleshooting](#troubleshooting)
   * [Encryption Error](#encryption-error)
- [FAQ - Frequently asked questions](#faq-frequently-asked-questions)
   * [**Q: Full MacOS / Linux Support?**](#q-full-macos-linux-support)
   * [**Q: How secure is the encrypted data within a game, and what measures can enhance its security?**](#q-how-secure-is-the-encrypted-data-within-a-game-and-what-measures-can-enhance-its-security)
- [Limited Editor Support for Linux and MacOS](#limited-editor-support-for-linux-and-macos)
- [Support](#support)
- [License](#license)

<!-- TOC end -->

![Final Factory Logo](https://static.wixstatic.com/media/880a29_adf69d1f5217420c946012af55973e12~mv2.png)  ![|150](https://static.wixstatic.com/media/880a29_92e86dd6a76b4276acf906e3423bb5d0~mv2.png)

[Final Preferences](https://u3d.as/3hny) is an advanced Unity asset created by Final Factory, designed to enhance and extend the management of preferences in Unity projects. It builds upon the existing PlayerPrefs and EditorPrefs frameworks, introducing additional functionalities such as ProjectRuntime and ProjectDevelopment scopes, encryption for sensitive data, and a sophisticated editor UI for managing preferences in real-time. This documentation provides detailed guidance on the setup, usage, features, and API of Final Preferences to ensure successful integration and utilization in your Unity projects.

## Introduction

Final Preferences delivers a robust and adaptable system for managing game settings and preferences across various scopes, including runtime and development standalone builds. This asset is particularly beneficial for projects that demand high data integrity and enhanced security, featuring encryption capabilities to protect sensitive information.

If you need further help, use the various ways to [[#Support|contact us]].
## Getting Started

### Installation

To begin using Final Preferences in your Unity project, follow these steps:

1. Purchase and download the Final Preferences asset from the Unity Asset Store.
2. Import the asset into your Unity project by navigating to `Window -> Package Manager`.

### Initial Setup

Final Preferences is designed for ease of use and does not require additional setup after importing into your project. It is ready to use immediately, enabling you to start enhancing your preference management system right away.

This streamlined approach ensures that developers can integrate Final Preferences quickly and efficiently, focusing on leveraging its features to improve the configuration and security of game preferences.

## Feature Overview

### Multi-Scope Preference Management

Final Preferences provides a versatile scope management system that accommodates various development and runtime scenarios in Unity. Each scope is uniquely designed with specific use cases, restrictions, and capabilities, enhancing how preferences are managed across different environments and builds. Here’s a detailed look at each available scope and their practical applications within Unity projects.

#### Player Scope

- **Description**: The Player Scope is used for preferences that need to be accessible both during gameplay and within the Unity Editor. These preferences are project-specific, meaning they are unique to each project but do not carry over across different projects or development environments. This scope utilizes Unity's PlayerPrefs as its backend.
- **Usability**: Ideal for storing player-specific settings such as volume levels, game difficulty, and graphical preferences. Preferences in this scope can be read and written during gameplay and while using the editor.
- **Restrictions**: Operates within the standard limitations of PlayerPrefs, with no additional restrictions.
- **Example Usage**: To save player settings that persist across gaming sessions, you can manage settings like volume as shown below:

```csharp
var playerVolume = new PreferenceItemFloat(PrefsScope.Player, "volumeLevel", 0.75f);
```

#### ProjectRuntime Scope

- **Description**: This scope is tailored for settings that need to remain consistent across various instances of the editor and runtime, and are accessible from any computer when shared via source control (using a Scriptable Object Asset). It is designed for project-specific settings that are crucial during development but should not be altered during actual gameplay.
- **Usability**: Settings in this scope can be modified in the editor but are readable in both the editor and during runtime, including in development and release builds.
- **Restrictions**: The settings are read-only during runtime and development builds, which prevents alterations during active gameplay.
- **Example Usage**: For settings that should not change during gameplay, such as API tokens (which can also be encrypted) or constants like points per game level:

```csharp
var spawnRate = new PreferenceItemString(PrefsScope.ProjectRuntime, "BackendGameAPIKey", "ABHEXD");
```

#### ProjectDevelopment Scope

- **Description**: This scope is reserved for settings relevant only during the development phase, ensuring they do not appear or become accessible in production builds. It supports access from any computer via source control, making it project-specific.
- **Usability**: Preferences within this scope are modifiable and readable in the editor and can be read in development builds.
- **Restrictions**: These settings are completely unavailable in production runtime, strictly for development use only.
- **Example**: Utilize this scope for debugging parameters that need adjustment during testing but should remain inaccessible in the final game, like debug display options.
- **Attention**: The `PrefsScope.ProjectDevelopment` is not present in release builds to prevent accidental inclusion of test code; it will trigger a build error. Use conditional compilation like `#if UNITY_EDITOR || DEVELOPMENT_BUILD` to safeguard against such issues.

```csharp
#if UNITY_EDITOR || DEVELOPMENT_BUILD
var debugDisplayEnabled = new PreferenceItemBool(PrefsScope.ProjectDevelopment, "debugDisplay", true);
#endif
```

#### Editor Scope

- **Description**: Tailored for preferences that are specific to a developer’s machine, ideal for configurations within the Unity Editor that should not be shared across environments or projects. This scope uses Unity's EditorPrefs as its backend.
- **Usability**: Preferences are both readable and writable within the editor and are confined to the developer’s computer.
- **Restrictions**: These settings do not impact the game runtime and are exclusively accessible within the editor.
- **Example**: Manage editor-specific settings like layout preferences or plugin configurations unique to a developer's workstation.
- **Attention**: The `PrefsScope.Editor` is available only within the Unity editor.

```csharp

#if UNITY_EDITOR
var preferredLayout = new PreferenceItemString(PrefsScope.Editor, "editorLayout", "Default");
#endif

```

<div style="page-break-after: always;"></div>


#### Standalone Scope

- **Description**: This scope is used for managing PlayerPrefs in standalone game builds that are run on a developer’s computer, allowing changes from the Unity Editor without needing to alter the game directly.
- **Usability**: Ideal for accessing or debugging PlayerPrefs from built applications, enabling modifications without the need for game rebuilds.
- **Restrictions**: Accessible only within the Unity Editor and not during actual gameplay or other development phases.
- **Example**: Adjust or reset PlayerPrefs settings in your standalone game to test different configurations effortlessly.
- **Attention**: `PrefsScope.Standalone` is operational solely within the Unity editor.

```csharp
#if UNITY_EDITOR
var standalonePlayerCoins = new PreferenceItemFloat(PrefsScope.Standalone, "Coins", 100, true, true);
#endif
```

Each of these scopes enhances the flexibility and control developers have over managing preferences, catering to specific needs and stages of game and application development with Unity.

### Encryption

Encrypting sensitive data is crucial for the security of user preferences. Final Preferences enables the encryption of crucial data such as credentials or personal user information, safeguarding it against unauthorized access and enhancing overall data integrity.

### Powerful Editor UI

Final Preferences comes equipped with a robust editor UI that greatly facilitates the visualization and real-time modification of preferences. This feature is particularly effective for managing both standard and encrypted data seamlessly.

- **Real-Time Visualization**: Adjust and monitor preferences on-the-fly within the Unity Editor, providing immediate feedback and control.
- **Standalone Game Build Support**: Directly view and modify preferences from standalone game builds, streamlining debugging and testing processes without the need to rebuild.

<div style="page-break-after: always;"></div>

### Preferences Item Framework

The Preferences Item Framework within Final Preferences utilizes a generic `PreferenceItem<T>` class. This framework enhances preference management through typed preference items, incorporating features that promote efficiency and ease of use:

- **Type-Specific Items**: Create and manage preferences with specific data types, ensuring data consistency and type safety.
- **Auto-Sync**: Automatically synchronize preference values between the item and the preferences handler, reducing the need for manual updates.
- **Change Events**: Trigger events when preferences change, enabling responsive and dynamic game behaviors based on user settings.
- **Default Values**: Define default values for each preference, simplifying the initialization process and ensuring a fallback when no user-defined value is present.
- **Watchable**: The Preferences Item includes a key feature that allows each preference item to be easily registered with the player preferences change monitor.

These features make Final Preferences a powerful tool for managing a wide array of settings in Unity projects, ensuring that developers can maintain control over user preferences with high levels of security and flexibility.

### Updating PlayerPrefs Values of Standalone Build in Realtime

Final Preferences enhances the debugging and testing process by allowing real-time updates and monitoring of PlayerPrefs values within standalone builds. This capability is provided through a sophisticated monitoring system that interacts with Unity's PlayerPrefs backend.

#### Real-Time Monitoring and Updates

You can select specific PlayerPrefs keys to monitor. The easiest way is to just call the method `Watch` on a `PreferenceItem<T>` . If the system detects any changes made outside of the game—such as through the editor—it triggers an event. This feature enables developers to easily modify and test different preferences values while the game is running, without the need to stop and restart the game, thus streamlining the debugging process.

<div style="page-break-after: always;"></div>

#### Using the `Watch` Method

To begin monitoring a PlayerPrefs key, you can utilize the `Watch` method directly on a `PreferenceItem<T>`. This method is designed for ease of use, allowing you to quickly set up monitoring for any preference item.

```csharp
// Example of setting up a watch on a preference item 
var playerVolume = new PreferenceItemFloat(PrefsScope.Player, "volumeLevel", 0.75f); 
playerVolume.Watch();
```

#### Integration and Control

The monitoring system is designed to be non-intrusive and is automatically excluded from release builds to prevent potential performance issues or unintended interactions in the final product. However, if there is a specific need to include this monitoring feature in a release build, Final Preferences provides a setting that allows developers to enable it manually. This flexibility ensures that developers can choose how and when to use this feature based on their specific needs and development stage.

This real-time update capability of PlayerPrefs values in standalone builds is an invaluable tool for developers looking to refine their game's settings dynamically and observe the immediate impacts of different configurations, all while maintaining high performance and security standards in production releases.

## Code Documentation

Every class and method is thoroughly documented using XML Documentation to ensure clarity and ease of use for developers.

## Code Concepts

### Prefs Class

The `Prefs` class is a static class that serves as a gateway to various preference handlers. It provides a method named `Get`, which returns an `IPrefs` instance. This instance acts as the handler for preferences within a specified `PrefsScope`.

### IPrefs Interface

The `IPrefs` interface provides a standardized approach to managing preferences within a particular scope. It ensures that preference operations are sensitive to the encryption state of the entries. A key point of this interface is its handling of duplicate keys: it does not allow a key to exist in both encrypted and non-encrypted forms simultaneously. This ensures integrity and security in the preference management system.

### PlayerPrefsChangeMonitor

The `PlayerPrefsChangeMonitor` is a specialized tool designed to monitor changes to the PlayerPrefs that occur outside the direct interaction of the game. This monitoring is typically active only in development builds, enhancing debugging and development efficiency. However, it can be explicitly enabled in release builds through a specific Project Setting.

This class provides methods `Watch` and `Unwatch` to register and deregister interest in specific keys. When a registered key is modified, the system notifies the relevant components, such as the Preferences Editor Window included with this package. This feature is particularly useful for developers needing real-time updates on preference changes during game development.

## Example Usage

### Using Preference Items

This example demonstrates how to define, access, and save a boolean preference item with encryption enabled. The preference item is created for the Player scope with a default value of `true` and then modified.

```csharp
// Define a new boolean preference item with encryption 
var myBoolPref = new PreferenceItemBool(PrefsScope.Player, "musicEnabled", true, true, encrypt: true);  
// Access and modify the value 
myBoolPref.Value = false;  
// Save the preference
myBoolPref.Save(); // Not necessary if autosave is enabled.
```

### Using Direct Code

Here, we illustrate how to directly retrieve and set a preference value within the Player scope. The methods provided by the `Prefs` class allow for straightforward preference management.

```csharp
// Getting the value
var myValue = Prefs.Get(PrefsScope.Player).GetString("MyValue", "Default Value");
// Setting the value
Prefs.Get(PrefsScope.Player).SetValue("MyValue", myValue);
```

These examples highlight the practical application of preference management within your application, ensuring easy retrieval and modification of user preferences.


<div style="page-break-after: always;"></div>

## Samples

### Encrypted Money Value

This example demonstrates how to utilize the Final Preferences asset for securely managing and storing player coin totals using encrypted preferences. It shows how to implement and modify encrypted coin values in real time, both through gameplay controls and via the Preferences Editor.

```csharp
private const string Key = "Coins";  
private const int DefaultValue = 100;  
  
public Text Text;  
public Button ButtonEncrypt;  
public Button ButtonDecrypt;  
  
private IPrefs _prefs;  
  
private bool _isEncrypted = true;  
  
// Easy access to the value  
private float Value  
{  
    get => _prefs.GetFloat(_isEncrypted, Key, DefaultValue);  
    set  
    {  
        _prefs.SetFloat(_isEncrypted, Key, value);  
        _prefs.TrySave();  
    }  
}  
  
// Getting the prefs instance  
private void Awake() => _prefs = Prefs.Get(PrefsScope.Player);  
  
private void Start()  
{  
    UpdateText();  
    UpdateButtonState();  
}  
  
private void OnEnable()  
{  
    // Register for changes.  
    _prefs.Changed += UpdateText;  
      
    // Watch for changes in the PlayerPrefs for any changes from the editor.  
    PlayerPrefsChangeMonitor.Watch(Key, _isEncrypted);  
}  
  
private void UpdateText(object sender, ChangePrefsEventArgs args)  
{  
    if (args.Key == Key)  
    {  
        UpdateText();  
    }  
}  
  
private void OnDisable()  
{  
    Prefs.Get(PrefsScope.Player).Changed -= UpdateText;  
    PlayerPrefsChangeMonitor.UnWatch(Key, _isEncrypted);  
}  
  
public void AddMoney()  
{  
    Value += 100;  
    UpdateText();  
}  
  
public void SpendMoney()  
{  
    Value -= 100;  
    UpdateText();  
}  
  
public void Encrypt()  
{  
    //Change the encryption state  
    _prefs.ChangeEncryptionState(Key, true);  
    _isEncrypted = true;  
    UpdateButtonState();  
}  
  
public void Decrypt()  
{  
    //Change the encryption state  
    _prefs.ChangeEncryptionState(Key, false);  
    _isEncrypted = false;  
    UpdateButtonState();  
}  
  
private void UpdateButtonState()  
{  
    ButtonEncrypt.interactable = !_isEncrypted;  
    ButtonDecrypt.interactable = _isEncrypted;  
}  
  
private void UpdateText() => Text.text = $"Coins: {Value}";
```

#### Real-Time Modifications

You can modify the coin values in real-time using the Preferences Editor by selecting the "Player" scope. If you build the scene as a development build, you can also modify these values when running the standalone build by selecting "Standalone" in the Preferences Editor.

### Encrypted Money Value Item

The **Encrypted Money Value Item** functions similarly to the **Encrypted Money Value** example but utilizes the `PreferenceItem<T>` class for a more straightforward and user-friendly approach to managing encrypted data. This method enhances ease of use while maintaining the same functionality.

```csharp
public Text Text;  
public Button ButtonEncrypt;  
public Button ButtonDecrypt;  
  
//Creating a new preference item for convenience access.  
private readonly PreferenceItemInt _moneyItem = new(PrefsScope.Player, "Coins", 100, true, true);  
  
private void Start()  
{  
    UpdateText();  
    UpdateButtonState();  
}  
  
private void OnEnable()  
{  
    //Register for changes and enable watching for changes from the editor.  
    _moneyItem.Changed += UpdateText;  
    _moneyItem.Watch();  
}  
  
private void OnDisable()  
{  
    _moneyItem.UnWatch();  
    _moneyItem.Changed -= UpdateText;  
}  
  
private void UpdateText(object sender, int e) => UpdateText();  
  
public void AddMoney()  
{  
    //Changing the value. This will automatically save the value because we enabled sync.  
    _moneyItem.Value += 100;  
    UpdateText();  
}  
  
public void SpendMoney()  
{  
    //Changing the value. This will automatically save the value because we enabled sync.  
    _moneyItem.Value -= 100;  
    UpdateText();  
}  
  
public void Encrypt()  
{  
    //change the encryption state  
    _moneyItem.IsEncrypted = true;  
    UpdateButtonState();  
}  
  
public void Decrypt()  
{  
    //change the encryption state  
    _moneyItem.IsEncrypted = false;  
    UpdateButtonState();  
}  
  
private void UpdateButtonState()  
{  
    ButtonEncrypt.interactable = !_moneyItem.IsEncrypted;  
    ButtonDecrypt.interactable = _moneyItem.IsEncrypted;  
}  
  
private void UpdateText() => Text.text = $"Coins: {_moneyItem.Value}";
```

Both methods ensure that the modification and management of sensitive data, such as player financial information, are handled securely and efficiently, providing developers with robust tools to enhance gameplay interaction and data security.


## Preferences Editor

The Preferences Editor is a fundamental component of Final Preferences, designed to facilitate the creation, encryption, and modification of preferences through an intuitive visual interface. This editor is feature-rich and user-friendly, enhancing your productivity and experience.

![Final Preferences Doc 2.png](https://static.wixstatic.com/media/880a29_2efedeb9d5314b0aa807308239872101~mv2.png)

### Toolbar

![Final Preferences Doc 3.png](https://static.wixstatic.com/media/880a29_732e757af81843d9a69e20df8f0830eb~mv2.png)

1. Quickly switch the selection to the tool specified in option #2.
2. Modify the scope currently displayed and editable.
3. Search for specific keys and their corresponding values.
4. Supports saving and loading, depending on the scope's capabilities.
5. Erase **ALL** entries within the current scope.
6. Access settings specific to the Preferences Editor.

### Preferences List

![Final Preferences Doc 4.png](https://static.wixstatic.com/media/880a29_b3e73d2ea6b04383a8d9e341e6c3c6f2~mv2.png)

1. Toggle the encryption status of an entry. A checked checkbox and a blue background indicate that the entry is currently encrypted, meaning it is accessible to the game only when an encrypted value is requested.
2. Display the key name.
3. Show the value associated with the key.
4. Indicate the current data type of the key, which can be changed by clicking on it. **Note: Changing the type to one that is incompatible with the current value will replace the existing value with a default value of the new type, resulting in the loss of the original value.**
5. Remove an individual entry.

### Adding New Entries

![Final Preferences Doc 5.png](https://static.wixstatic.com/media/880a29_502e5ef8fc404b2990cc1c9dc24fd119~mv2.png)

This section at the end of the list allows for the addition of new entries. The process mirrors the existing list, except that it features an 'Add' button instead of a 'Remove' button to facilitate the inclusion of new preferences.

## Troubleshooting

Some common issues and their solutions.
### Encryption Error

![Final Preferences Doc 1.png](https://static.wixstatic.com/media/880a29_286600c5b75045a99cec970b46f97465~mv2.png)

It is like that the encryption key is missing or wrong. Restore a backup to retrieve the data or if you do not need the data anymore, you can just delete the key.

## FAQ - Frequently asked questions

### **Q: Full MacOS / Linux Support?**

**A:** Yes, but not for the Editor. Support for MacOS and Linux is currently limited. The Preferences Editor and the Monitor features do not function on MacOS and Linux; however, all other functionalities of Final Preferences operate as expected. Read more: [[#Limited Editor Support for Linux and MacOS]]
### **Q: How secure is the encrypted data within a game, and what measures can enhance its security?**

**A:** The encryption system used in Final Preferences employs AES (Advanced Encryption Standard), which is a robust and commonly used encryption method. This level of encryption can effectively prevent easy or casual access to sensitive data stored within the game preferences. However, it's important to recognize that no encryption solution on the client side can guarantee 100% security.

We store the encryption key within a ScriptableObject, which makes it more difficult (but not impossible) for unauthorized users to access or modify. Despite these measures, skilled individuals with knowledge in reverse engineering could potentially decrypt the data by extracting game assets and analyzing them to find the encryption key.

To enhance the security of your data further, consider the following additional measures:

1. **Use IL2CPP:** This Unity engine feature converts managed code into C++, which adds an extra layer of security by making it harder to reverse engineer the code compared to traditional Mono scripting backend.

2. **Employ Obfuscators:** Code obfuscation is a technique that makes the source code much harder to read and understand. When the code is obfuscated, it complicates the reverse engineering process, thus increasing the security against tampering and exploitation.

3. **Regular Updates:** Regularly updating the encryption algorithms and the methods used to secure the encryption keys can help in maintaining a higher level of security.

4. **Minimal Sensitive Data:** As a best practice, minimize the amount of sensitive data stored on the client side. Where possible, sensitive information should be managed server-side where higher security measures can be implemented.

For a client-side solution, Final Preferences offers a fairly secure system. The combination of AES encryption, storing keys in a less accessible manner, and the potential addition of IL2CPP and obfuscation techniques provides a reasonable level of resistance to unauthorized access. Nonetheless, it is crucial to maintain awareness that no client-side security measure is uncrackable and should be supplemented with best practices in data handling and security.

## Limited Editor Support for Linux and MacOS

The limited editor support for Linux and MacOS is in place for an undefined period. This limitation specifically affects the Preferences UI Editor, where editing capabilities for PlayerPrefs, EditorPrefs, and Standalone Prefs are unavailable on these platforms. However, ProjectRuntime and ProjectDevelopment preferences function correctly. Additionally, the runtime PlayerPrefs Change Monitor feature does not support Linux and MacOS. 

**It's important to note that while this limitation exists within the editor environment only, all preferences including Prefs, PlayerPrefs, EditorPrefs, ProjectRuntime, and ProjectDevelopment are fully operational on any platform during runtime.**

<div style="page-break-after: always;"></div>

## Support

If you need help or have any questions, please contact our support at:

* GitHub: [Final Preferences Issues](https://github.com/FinalFactory/FinalPreferences/issues)
- Forum: [Discussions](https://github.com/FinalFactory/FinalPreferences/discussions)
- Discord: [Final Factory Discord Server](https://finalfactory.de/discord)
- Email: support@finalfactory.de
* Youtube: [Final Factory Youtube Channel](https://www.youtube.com/@final-factory)

## License

Released under the [Unity Asset Terms](https://unity.com/legal/as-terms)
Copyright © 2024 Final Factory


Documentation Version 1.0
