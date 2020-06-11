# Keyple Calypso Application API
version 0.9 (current 'develop' branch)

## Calypso Portable Object Selection
Compared to the generic SE Selection API (cf. https://calypsonet.github.io/keyple/KeypleCoreApi/ApplicationApi/#se-selection), a PO Selector could be defined to accept only non-invalidated Portable Object (in this cas an invalidated PO isn't selected).

In addition, a PO Selection Request provides methods:

 - to prepare Select File command (useful in particular to manage REV1 Calypso PO for which the select of the targeted DF is required).
 - and to prepare simple read record command (useful to optimize the read of a file present on all targeted PO).

The matching SE resulting from a PO Selection Request is a Calypso PO. In case file records have been read during the selection: the corresponding data could be recovered in the Calypso PO card image.

![Calypso Selection v0.9](img/KeypleCalypso_ApplicationApi_ClassDiag_Transaction_PO_Selection_0_9_0.svg)

## Calypso Portable Object Transaction

A Secure Element resource (SeResource) is a set of a smart card reader and a **selected** secure element application.

 - A Calypso Portable Object (CalypsoPo) is the image of a selected Calypso PO.
 - A Calypso SAM (CalypsoSam) is the image of a selected Calypso SAM.

To operate a Calypso transaction:

 - At least a Calypso resource (SeResource<CalypsoPo>) is required.
 - A SAM resource ((SeResource<CalypsoSam>) is required too if security features are involved (Calypso secure session, Stored value transaction, PIN encryption, etc…).

A Calypso PO image provides public ‘getters’ in order to **recover** the information of the selected PO (startup data, file data, … etc).

A transaction with a Calypso PO is fully managed through the PoTransaction object:

 - First a set of PO commands could be defined through ‘**prepare**’ commands.
 - Next the prepared PO commands transmitted when operating a ‘**process**’ command.
 - The responses of the PO are then recovered through the Calypso PO image.

![Global Architecture](../../SecureElementTerminalApi/CalypsoApi/img/CalypsoTerminal_ApplicationApi_ClassDiag_Transaction_Global.svg)

### Calypso card image

![Calypso PO card image](img/KeypleCalypso_ApplicationApi_ClassDiag_Transaction_CalypsoPo_0_9_0.svg)

### Calypso transaction

Most of the process methods have a ‘Channel Control’ parameter in order to define if the logical with the selected Calypso has to be kept open or to be closed after the processing of the prepared PO commands.
 - processPoCommands is used to transmit a set of prepared PO commands outside of a secure session.
 - processOpening issues an Open Secure Session followed by the prepared PO commands.
 - processPoCommandsInSession allows to transmit a set of prepared PO commands inside of a secure session.
 - processClosing issues first the last prepared PO commands and transmits a Close Secure Session.
 - prepareManageSession allows to change authenticate or change the encryption mode.

![Calypso transaction](img/KeypleCalypso_ApplicationApi_ClassDiag_Transaction_PoTransaction_0_9_0.svg)

## Data model extension

![Calypso transaction](img/KeypleCalypso_ApplicationApi_ClassDiag_Transaction_SpecificPoTransaction_0_9_0.svg)
