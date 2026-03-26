# Requirements Document

## Introduction

A mobile application designed to enhance personal safety for women. The app provides a prominent SOS button that instantly sends SMS alerts with GPS coordinates to pre-configured trusted contacts. It also supports real-time live location sharing and a map view of nearby safe places such as police stations and hospitals.

## Glossary

- **App**: The Women's Safety mobile application
- **User**: The person using the App on their mobile device
- **SOS_Button**: The large, prominently displayed emergency trigger button on the home screen
- **Alert**: An SMS message containing the User's current GPS coordinates and a distress message
- **Trusted_Contact**: A phone number and name saved by the User to receive Alerts and live location updates
- **Contact_Manager**: The App component responsible for adding, editing, and removing Trusted_Contacts
- **GPS_Service**: The device component that provides the User's current geographic coordinates
- **SMS_Service**: The device component responsible for sending SMS messages
- **Live_Tracking**: A feature that continuously shares the User's real-time location with Trusted_Contacts
- **Location_Tracker**: The App component that manages Live_Tracking sessions
- **Safe_Places_Map**: The in-app map view displaying nearby police stations and hospitals
- **Places_Service**: The external service queried to find nearby safe locations

---

## Requirements

### Requirement 1: SOS Alert

**User Story:** As a User in distress, I want to press a single large button to immediately alert my Trusted_Contacts with my location, so that help can be dispatched to me quickly.

#### Acceptance Criteria

1. THE App SHALL display the SOS_Button prominently on the home screen at all times.
2. WHEN the User presses the SOS_Button, THE App SHALL retrieve the User's current coordinates from the GPS_Service within 5 seconds.
3. WHEN the GPS_Service returns coordinates, THE SMS_Service SHALL send an Alert to every saved Trusted_Contact containing the User's name, GPS coordinates, and a Google Maps link to those coordinates.
4. WHEN the SOS_Button is pressed and no Trusted_Contacts are saved, THE App SHALL display a message prompting the User to add at least one Trusted_Contact before sending an Alert.
5. IF the GPS_Service fails to return coordinates within 5 seconds, THEN THE App SHALL send the Alert with the last known coordinates and indicate in the message that the location may be outdated.
6. IF the SMS_Service fails to deliver an Alert to a Trusted_Contact, THEN THE App SHALL display a notification listing which Trusted_Contacts did not receive the Alert.
7. WHEN an Alert is sent, THE App SHALL display a confirmation screen showing the timestamp and the list of Trusted_Contacts who were notified.

---

### Requirement 2: Trusted Contacts Management

**User Story:** As a User, I want to add and remove trusted people who will receive my SOS alerts, so that only the right people are notified in an emergency.

#### Acceptance Criteria

1. THE Contact_Manager SHALL allow the User to add a Trusted_Contact by providing a name and a valid phone number.
2. WHEN the User submits a new Trusted_Contact, THE Contact_Manager SHALL validate that the phone number conforms to a valid international or local format before saving.
3. IF the User submits a phone number that is already saved as a Trusted_Contact, THEN THE Contact_Manager SHALL reject the duplicate and display an error message.
4. THE Contact_Manager SHALL allow the User to remove any saved Trusted_Contact.
5. WHEN a Trusted_Contact is removed, THE App SHALL no longer include that contact in future Alerts or Live_Tracking sessions.
6. THE Contact_Manager SHALL allow the User to import contacts directly from the device's native contact list.
7. THE App SHALL persist Trusted_Contacts across app restarts.

---

### Requirement 3: Live Tracking

**User Story:** As a User, I want to share my real-time location with my Trusted_Contacts, so that they can monitor my movement and respond if I stop moving unexpectedly.

#### Acceptance Criteria

1. THE App SHALL display a Live_Tracking toggle on the home screen.
2. WHEN the User enables the Live_Tracking toggle, THE Location_Tracker SHALL begin transmitting the User's GPS coordinates to all saved Trusted_Contacts at intervals no greater than 30 seconds.
3. WHEN the User disables the Live_Tracking toggle, THE Location_Tracker SHALL stop transmitting location updates and notify Trusted_Contacts that the session has ended.
4. WHILE Live_Tracking is active, THE App SHALL display a visible indicator on the home screen showing that location sharing is in progress.
5. WHILE Live_Tracking is active and the App is moved to the background, THE Location_Tracker SHALL continue transmitting location updates.
6. IF the GPS_Service becomes unavailable while Live_Tracking is active, THEN THE Location_Tracker SHALL notify Trusted_Contacts that location updates are temporarily unavailable and resume transmission when the GPS_Service becomes available again.
7. WHEN the User enables Live_Tracking and no Trusted_Contacts are saved, THE App SHALL display a message prompting the User to add at least one Trusted_Contact.

---

### Requirement 4: Safe Places Map

**User Story:** As a User, I want to see nearby police stations and hospitals on a map, so that I can quickly navigate to a safe location.

#### Acceptance Criteria

1. THE App SHALL provide a Safe_Places_Map screen accessible from the home screen.
2. WHEN the Safe_Places_Map screen is opened, THE App SHALL request the User's current location from the GPS_Service and display it on the map.
3. WHEN the User's location is available, THE Places_Service SHALL be queried for police stations and hospitals within a 5 km radius of the User's current coordinates.
4. WHEN the Places_Service returns results, THE Safe_Places_Map SHALL render each result as a distinct map marker indicating its category (police station or hospital).
5. WHEN the User taps a map marker, THE Safe_Places_Map SHALL display the place's name, address, and distance from the User's current location.
6. WHEN the User taps a map marker's detail view, THE App SHALL open the device's native navigation app with directions to that location.
7. IF the Places_Service returns no results within the 5 km radius, THEN THE Safe_Places_Map SHALL display a message informing the User that no safe places were found nearby and offer to expand the search radius to 10 km.
8. IF the GPS_Service is unavailable when the Safe_Places_Map is opened, THEN THE App SHALL display an error message and provide a prompt to enable location permissions.

---

### Requirement 5: GPS Location Access

**User Story:** As a User, I want the app to access my device's GPS, so that my location can be included in alerts and shown on the map.

#### Acceptance Criteria

1. WHEN the App is launched for the first time, THE App SHALL request location permission from the User before any GPS_Service access is attempted.
2. WHEN the User grants location permission, THE GPS_Service SHALL be available for use by the SOS_Button, Location_Tracker, and Safe_Places_Map.
3. IF the User denies location permission, THEN THE App SHALL display an explanation of why location access is required and provide a prompt to open device settings to grant the permission.
4. WHILE Live_Tracking is active, THE App SHALL request background location permission on platforms that require it separately.
5. THE App SHALL request only the minimum location permission scope required for each feature.

---

### Requirement 6: SMS Sending Capability

**User Story:** As a User, I want the app to send SMS messages on my behalf, so that my Trusted_Contacts are notified even if they do not have the app installed.

#### Acceptance Criteria

1. WHEN the App is launched for the first time, THE App SHALL request SMS permission from the User before any SMS_Service access is attempted.
2. WHEN the User grants SMS permission, THE SMS_Service SHALL be available to send Alerts to Trusted_Contacts.
3. IF the User denies SMS permission, THEN THE App SHALL display an explanation of why SMS access is required and provide a prompt to open device settings to grant the permission.
4. WHEN an Alert is sent, THE SMS_Service SHALL send one SMS per Trusted_Contact containing the distress message and location link.
5. THE App SHALL not send SMS messages for any purpose other than Alerts triggered by the User.
