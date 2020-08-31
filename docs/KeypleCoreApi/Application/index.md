# Keyple Core Application API
version 0.9 (current 'develop' branch)

## Reader Access
On Keyple, the smart card readers are managed through plugins in order to integrate specific reader solutions.
The '**Secure Element Proxy Service**' singleton provides the unique name list of registered plugins. There can be three kinds of plugin:
 - The ‘**Reader Plugin**’ is the generic interface to list the readers of a plugin, or to access to a specific reader with its name.
 - The ‘**Observable Plugin**’ interface extends reader plugins which have the capability to be observed: in order to notify registered Plugin Observers about the plug or unplug of readers. Plugin observers could be added or removed to the observable plugin. Useful for systems allowing the hot plug / unplug of readers.
 - A ‘**Reader Pool Plugin**’ is a plugin for which a reader is available only after an explicit allocation. When not more necessary, a reader must be released. Useful for server solutions managing farms of readers or interfaced with HSM: unallocated readers or HSM SE instances could be shared between several smart card terminal solutions.

A smart card reader is identified through its unique name in a plugin. There are two kinds of reader:
 - The ‘**SE Reader**’ is the generic interface to handle a smart card reader. The presence of SE in a reader could be checked.
 - The ‘**Observable Reader**’ interface extends SE readers which have the capability to notify registered Reader Observers about the insertion or remove of a Secure Element in the reader. Reader observers could be added or removed to the observable reader. Useful for systems automatically starting the processing of a SE at its insertion: like a ticketing validator.
![Reader Access v0.9](img/KeypleCore_ApplicationApi_ClassDiag_SE_Proxy_PluginSettingAndReaderAccess_0_9_0.svg)

(The APDU transmission with a Secure Element is managed at the low-level SE Channel API.)

### Specific Plugin
To hide plugin native implementation classes, the reader plugins are registered to the SE Proxy Service through related specific plugin factory.
![Specific Plugin v0.9](img/KeypleCore_ApplicationApi_ClassDiag_SE_Proxy_SpecificPluginAndReader_0_9_0.svg)

### Reader Notifications
To be notified about '**Plugin Event**' or '**Reader Event**', a terminal application must implement the dedicated '**Plugin Observer**' or '**Reader Observer**' interfaces.

![Reader Notifications v0.9.0](img/KeypleCore_ApplicationApi_ClassDiag_SE_Proxy_ObservablePluginAndReaderEvents_0_9_0.svg)

#### Plugin event
Several ‘Plugin Observers’ could be registered to an Observable Plugin.
In case of reader connection / disconnection, the observable plugin notifies sequentially the registered observers with the corresponding plugin event.
The observable plugin is a blocking API, the thread managing the issuance of the plugin event waits the acknowledge of the observer currently notified.

#### Reader event
Several ‘Reader Observers’ could be registered to an Observable Reader.
In case of SE insertion / match / removal, the observable reader notifies sequentially the registered observers with the corresponding reader event. The observable reader could be a blocking API, the thread managing the issuance of the plugin event could wait the acknowledge of the notified observers.

An observable reader has the capability to be set with a ‘Default Selections Request’: in this case when a SE is inserted in the reader, the reader will try to operate the configured default selections. If a selection successfully matches with the SE, instead to simply notify the insertion of SE, the observable reader will notify about a successful selection with a SE application.
 - If the notification mode is defined as ‘always’, then in case of SE insertion, the observable reader will notify a SE matched reader event in case of successful selection, or a simple SE inserted reader event if not.
 - If the notification mode is defined as ‘matched only’, then in case of SE insertion, simple SE inserted reader events are not notified.

When the processing of an inserted or matched SE is finished, a reader observer must release the logical channel with the Secure Element, in order to prepare the observable reader to detect the removal of the SE.

#### Observable reader states
An observable reader is active only when at least one reader observer is registered, and if the start of the detection has been requested. 
When active, an observable read could switch between three internal states: ‘Wait for SE Insertion’, ‘Wait for SE Processing’, & ‘Wait for SE Removal’.

In the nominal case, a Reader Observer indicates to the observable reader that the processing of the SE is finished by releasing the SE Channel.
To manage a failure of the reader observer process, the observable reader interface provides also a method to finalize the SE processing.

![Observable Reader States](img/KeypleCore_ApplicationApi_StateDiag_SE_Proxy_ObservableReaderStates.svg)

The states could be switched:
 - due to an explicit API request (blue arrows):
   - the release of the Secure Element Channel,
   - the call of an Observable Reader method:
     - the addition or the remove of an Observable Reader,
     - a request to start or stop the detection, to finalize the SE processing.
 - Or because of an external event (red arrows), the insertion or the remove of a SE.
   - the insertion a SE causing the observable reader to notify a 'SE matched' reader event (in case of successful
   l default selection) or a 'SE inserted' reader event (Notification Mode defined as always).
   - the removal of a SE causing the observable reader to notify a 'SE removed' reader event.

If a SE detection is started with the 'repeating' polling mode, then later when the SE is removed, the reader starts again to detect a SE.

Whatever the plugin of observable reader, when waiting for the SE removal, any observable reader shall have the capability to notify the remove of the SE.
Some reader plugin solution could have the capability to notify a SE removal also during the processing of the SE.


## SE Selection

### Selection scenarios
Depending on the SE transaction use case, or on the reader capability, there are two ways to manage the selection of a SE:
 - Either on a SE reader, a selection could be operated directly by transmitting the selection request. In this case the same entity manages both the SE selection and the SE processing.
 - Otherwise, on an Observable Reader, a default selection could be defined. In this case the selection is operated automatically at the insertion of the SE. In this case, the SE selection is next managed by the observable reader, but the SE processing is managed by a reader observer.

![Selection v0.9](img/KeypleCore_ApplicationApi_ActivityDiag_Selection_Scenarii.svg)

### Selection setting and processing
A SE Selection request is defined with a SE Selector. A SE Selector could be defined with tree optional levels of selection filter.
 - The selection could be limited to match a specific SE communication protocol.
 - The SE ATR could be filtered to match a regular expression.
 - If an AID is defined, the local reader transmits a Select Application APDU command to the SE.
If a SE Selector is defined without any filter, the selection is always successful if a SE is present in the reader.

Depending on the Keyple SE extension library, a SE request could be completed with specific SE commands to operate at the selection (for example, a Select File for a specific DF LID, the read of a specific file).

For terminal managing several kinds of SE applications, a SE Selection could be prepared with several SE selection request to operate sequentially with the SE.

According to the defined 'multi SE request processing' mode, the SE selection could stop at the first selection request matching SE application, otherwise all the prepared SE selection request could be operated.
 - Before the new processing of SE selection request, the logical channel previously opened is closed.
 - The 'channel control' defines if the logical channel should be kept open or close after the last processed SE selection request.

![SE Selection v0.9](img/KeypleCore_ApplicationApi_ClassDiag_Selection_SelectorAndSelection_0_9_0.svg)

The result of a SE request selection is a card image of a matching SE. For a SE selection with multiple requests, several matching SE could be provided.
