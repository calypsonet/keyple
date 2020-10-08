# Keyple Core - Interface for the development of ticketing terminal application
version 1.0.0

## Reader Access
On Keyple, the smart card readers are managed through plugins in order to integrate specific reader solutions.
The '**SmartCard Service**' singleton provides the unique name list of registered plugins. There can be three kinds of plugin:
 - The ‘**Reader Plugin**’ is the generic interface to list the readers of a plugin, or to access to a specific reader with its name.
 - The ‘**Observable Plugin**’ interface extends reader plugins which have the capability to be observed: in order to notify registered Plugin Observers about the plug or unplug of readers. Plugin observers could be added or removed to the observable plugin. Useful for systems allowing the hot plug / unplug of readers.
 - A ‘**Reader Pool Plugin**’ is a plugin for which a reader is available only after an explicit allocation. When not more necessary, a reader must be released. Useful for server solutions managing farms of readers or interfaced with HSM: unallocated readers or HSM instances could be shared between several smart card terminal solutions.

A smart card reader is identified through its unique name in a plugin. There are two kinds of reader:
 - The ‘**Card Reader**’ is the generic interface to handle a smart card reader. The presence of card in a reader could be checked.
 - The ‘**Observable Reader**’ interface extends card readers which have the capability to notify registered Reader Observers about the insertion or remove of a smartcard in the reader. Reader observers could be added or removed to the observable reader. Useful for systems automatically starting the processing of a card at its insertion: like a ticketing validator.
![Reader Access v1.0.0](img/KeypleCore_Reader_ClassDiag_PluginSettingAndReaderAccess_1_0_0.svg)

(The APDU transmission with a card is managed at a lower layer, throught the SmartCard solution API.)

### Specific Plugin
To hide plugin native implementation classes, the reader plugins are registered to the Card Service through related specific plugin factory.
![Specific Plugin v1.0.0](img/KeypleCore_Reader_ClassDiag_SpecificPluginAndReader_1_0_0.svg)

### Reader Notifications
To be notified about '**Plugin Event**' or '**Reader Event**', a terminal application must implement the dedicated '**Plugin Observer**' or '**Reader Observer**' interfaces.

![Reader Notifications v1.0.0](img/KeypleCore_Reader_ClassDiag_ObservablePluginAndReaderEvents_1_0_0.svg)

#### Plugin event
Several ‘Plugin Observers’ could be registered to an Observable Plugin.
In case of reader connection / disconnection, the observable plugin notifies sequentially the registered observers with the corresponding plugin event.
The observable plugin is a blocking API, the thread managing the issuance of the plugin event waits the acknowledge of the observer currently notified.

#### Reader event
Several ‘Reader Observers’ could be registered to an Observable Reader.
In case of card insertion / removal or selection match, the observable reader notifies sequentially the registered observers with the corresponding reader event. The observable reader could be a blocking API, the thread managing the issuance of the plugin event could wait the acknowledge of the notified observers.

An observable reader has the capability to be set with a ‘Default Selections Request’: in this case when a SE is inserted in the reader, the reader will try to operate the configured default selections. If a selection successfully matches with the SE, instead to simply notify the insertion of SE, the observable reader will notify about a successful selection with a SE application.
 - If the notification mode is defined as ‘always’, then in case of card insertion, the observable reader will notify a matched card reader event in case of successful selection, or a simple inserted card reader event if not.
 - If the notification mode is defined as ‘matched only’, then in case of card insertion, simple inserted card reader events are not notified.

When the processing of an inserted or matched card is finished, a reader observer must release the logical channel with the smartcard, in order to prepare the observable reader to detect the removal of the card.

#### Observable reader states
An observable reader is active only when at least one reader observer is registered, and if the start of the detection has been requested. 
When active, an observable read could switch between three internal states: ‘Wait for Card Insertion’, ‘Wait for Card Processing’, & ‘Wait for Card Removal’.

In the nominal case, a Reader Observer indicates to the observable reader that the processing of the SE is finished by releasing the Card Channel.
To manage a failure of the reader observer process, the observable reader interface provides also a method to finalize the card processing.

![Observable Reader States](img/KeypleCore_Reader_StateDiag_ObservableReaderStates_1_0_0.svg)

The states could be switched:
 - due to an explicit API request (blue arrows):
   - the release of the Card Channel,
   - the call of an Observable Reader method:
     - the addition or the remove of an Observable Reader,
     - a request to start or stop the detection, to finalize the card processing.
 - Or because of an external event (red arrows), the insertion or the remove of a card.
   - the insertion a card causing the observable reader to notify a 'card matched' reader event (in case of successful default selection) or a 'card inserted' reader event (Notification Mode defined as always).
   - the removal of a card causing the observable reader to notify a 'card removed' reader event.

If a card detection is started with the 'repeating' polling mode, then later when the card is removed, the reader starts again to detect a card.

Whatever the plugin of observable reader, when waiting for the card removal, any observable reader shall have the capability to notify the remove of the card.
Some reader plugin solution could have the capability to notify a card removal also during the processing of the card.


## Card Selection

### Selection scenarios
Depending on the card transaction use case, or on the reader capability, there are two ways to manage the selection of a card:
 - Either on a card reader, a selection could be operated directly by transmitting the selection request. In this case the same entity manages both the card selection and the card processing.
 - Otherwise, on an Observable Reader, a default selection could be defined. In this case the selection is operated automatically at the insertion of the card. In this case, the card selection is next managed by the observable reader, but the card processing is managed by a reader observer.

![Selection v1.0.0](img/KeypleCore_CardSelection_ActivityDiag_Scenarii.svg)

### Selection setting and processing
A Card Selection request is defined with a Card Selector. A Card Selector could be defined with tree optional levels of selection filter.
 - The selection could be limited to match a specific card communication protocol.
 - The Card ATR could be filtered to match a regular expression.
 - If an AID is defined, the local reader transmits a Select Application APDU command to the card.
If a SE Selector is defined without any filter, the selection is always successful if a card is present in the reader.

Depending on the Keyple Card Solution extension library, a card request could be completed with specific card commands to operate at the selection (for example, a Select File for a specific DF LID, the read of a specific file).

For terminal managing several kinds of card applications, a Card Selection could be prepared with several card selection request to operate sequentially with the card.

According to the defined 'multi selection processing' mode, the card selection could stop at the first selection request matching card application, otherwise all the prepared card selection request could be operated.
 - Before the new processing of card selection request, the logical channel previously opened is closed.
 - The 'channel control' defines if the logical channel should be kept open or close after the last processed card selection request.

![Card Selection v1.0.0](img/KeypleCore_CardSelection_ClassDiag_SelectorAndSelection_1_0_0.svg)

The result of a card request selection is a card image of a matching card. For a card selection with multiple requests, several matching card could be provided.
