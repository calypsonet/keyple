# Keyple Core - 'Plugin API': for the implementation of smartcard reader plugins
version 1.0.0

## Plugin Factorized Processing

The implementation of Plugins requires to extend the classes AbstractPlugin (or AbstractThreadedObservablePlugin is the reader solution allows the hot plug/unplug of readers) and AbstractLocalReader (or AbstractObservableLocalReader if the reader has the capability to detect the insertion or the removal or a card)..
Only the abstract methods indicated in blue have to be implemented have to be implemented by the specific plugins.

For plugins with ObservableReader: depending on the capability of the reader solution different interfaces could be implemented:

 - SmartInsertionReader if the reader solution directly manages the card insertion listening,
 - NativeInsertionReader in case the reader solution is notified by the operating system in case of card insertion,
 - SmartRemovalReader for reader solutions which have the capability to detect the card removal without sending APDU commands.

![Plugin Factorized Processing v1.0.0](img/KeypleCore_Plugin_ClassDiag_PluginImplementaion_1_0_0.svg)

