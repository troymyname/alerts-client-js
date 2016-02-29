# alerts-management-client

## Javascript library for managing Barchart alerts

The Barchart Alert system provides notifications (e.g. SMS messages, emails, etc) to
Barchart users when user-defined events occur. These events are typically based on
market date (e.g. when a stock trades in excess of a given price).

This library allows a user to create, delete, and otherwise manage their alerts.


##Setup

This library is intended for use in the browser or headless JavaScript environments
(i.e. Node.js). It has been released via the usual package managers.


###npm

To import the library as a dependency to your application using npm, use the following command:


	npm install alerts-management-client -S


###bower

To import the library as a dependency to your application using bower, use the following command:


	bower install alerts-management-client -S


###git

The git repository is publicly-accessible here:


	https://github.com/barchart/alerts-client-js


##Usage

The Barchart Alert Management system exposes both REST and socket.io[http://socket.io] endpoints.
However, this library is a convenience-wrapper for interacting with the Barchart Alert
Management system. So, instead of invoking these transports directly, and worrying about the
details of the protocol, the consumers can make single-line method calls to perform asynchronous
operations (e.g. create alert, delete alert, etc).


###Example

An working example browser implementation can be found in the git repository at:


	/example/browser/example.html


Clone the git repository and open the HTML page in a browser. The consumer
code is contained within a script block of the HTML.


###Initialization (Using REST)

In the browser, an object has been added to the global namespace. Connect as follows:


	var alertManager = new Barchart.Alerts.RestAlertManager();


Alternately, the URL and port of the alert management system can be passed to the
constructor:


	var alertManager = new Barchart.Alerts.RestAlertManager('alerts-management-stage.elasticbeanstalk.com', 80);


Then, call the connect method before using any operations:

	alertManager.connect()
		.then(function() {
			// ready
		});


###Initialization (Using Socket.IO)

To use a Socket.IO transport, change the constructor to:

	var alertManager = new Barchart.Alerts.SocketIOAlertManager();

Specify the URL and port as follows:


	var alertManager = new Barchart.Alerts.SocketIOAlertManager('alerts-management-stage.elasticbeanstalk.com', 80);


And, finally, call the connect method before invoking any other operations:

	alertManager.connect()
		.then(function() {
			// ready
		});


###Asynchronous Operations

All operations are asynchronous. The result of any method call will be an A+ style promise. Here
is an example:


	alertManager.getServerVersion()
		.then(function(version) {
			console.log(version.semver);
		})
		.catch(function(error) {
			console.log('A problem occurred.');
		});

##Data

The library uses a JSON-in, JSON-out architecture. All data exchanged is in JSON format.
Requests must supply JSON data. Responses are composed of JSON data.

The following data structures are important:

###Target

A "target" refers to a type of object that can be observed.

    {
        "target_id": 1,
        "description": "equity",
        "identifier_type": "symbol",
        "identifier_description": "symbol"
    }


###Property

A "property" refers to an attribute of a target. The value of
a "property" can be checked using an "operator" object.

    {
        "property_id": 18,
        "description": [
            "Average Volume",
            "200 Day"
        ],
        "group": "Technical",
        "target": {
            "target_id": 1,
            "description": "equity",
            "identifier_description": "symbol"
        },
        "valid_operators": [
            2,
            3
        ]
    }

The "valid_operators" array references the "operator" objects
that can be used in conjunction with this "property" to
create a "condition." So, refer to the example above. This
"property" describes the 200-day average volume for an equity.
The  "valid_operators" array tells us that operators 2 and 3
(greater-than and less-than) can be used to build an
alert "condition" however it would not be valid to pair any
other operators with this "property."


###Operator

A comparison operation that can be applied to the "property" value
of a "target" object.

    {
        "operator_id": 4,
        "operator_name": "is-indicator",
        "display": {
            "short": "=",
            "long": "is"
        },
        "operand_type": "string",
        "operand_options": [
            "Buy",
            "Sell",
            "Hold"
        ]
    }

The operator.operand_options lists the possible values which can be used
as an "operand" when using the operator. If operator.operand_options has
no items, then there no restriction is placed upon an "operand" value.


###Condition

A "condition" is the comparison between a "property" value using an "operator" object.
The "target" of the "property" must include an "identifier" and the "operator" must
include an "operand" value. Alerts can have multiple conditions.

    {
        "condition_id": "0ce4e213-8fbf-442a-a3f5-2356a5eca09f",
        "property": {
            "property_id": 10,
            "description": [
                "Moving Average",
                "50 Day"
            ],
            "group": "Technical",
            "target": {
                    "target_id": 1,
                    "description": "equity",
                    "identifier_description": "symbol",
                    "identifier": "TSLA"
            }
        },
        "operator": {
            "operator_id": 2,
            "operator_name": "greater-than",
            "display": {
                "short": ">",
                "long": "greater than"
            },
            "operand_type": "number",
            "operand": "200"
        }
    }


###PublisherType

A "publisher type" defines a mechanism for notifying users.

    {
        "publisher_type_id": 1,
        "transport": "sms",
        "provider": "twilio"
    }


###PublisherTypeDefault

A "publisher type" that includes the default recipient for a user.

    {
        "publisher_type_id": 1,
        "transport": "sms",
        "provider": "twilio",
		"alert_system": "barchart.com",
		"user_id": "barchart-test-user",
		"default_recipient": "123-456-7890",
		"active_alert_types": [
			"price",
			"news"
		]
    }


###Publisher

A "publisher" defines the rules for notification of an end user. An
alert can have multiple publishers.

    {
        "publisher_id": "9c864a19-ce77-4a87-8cd6-e0810ecb120e",
        "use_default_recipient" false,
        "recipient": "123-456-7890",
        "format": "It is a good time to buy Telsa stock.",
        "type": {
            "publisher_type_id": 1,
            "transport": "sms",
            "provider": "twilio"
        }
    }


###Alert

An "alert" consists of one or more "condition" objects and one or more
"publisher" objects.

    {
        "alert_id": "39b633bf-8993-491d-b544-bdc9deed60be",
        "alert_state": "Inactive",
        "alert_system": "barchart.com",
        "user_id": "barchart-test-user",
        "alert_type": "price",
        "name": "Buy TSLA",
        "user_notes": "Time to buy Tesla Motors stock",
        "automatic_reset": true,
        "create_date": "1453673000873",
        "last_trigger_date": "145367399999",
        "conditions": [ ],
        "publishers": [ ]
    }

* If the alert has never been triggered, the "last_trigger_date" property will be omitted.
* The "alert_type" is an optional field that is used to classify the alert. It is used to decide "default" publishing rules (if no publishers have been specified). This happens by matching the "active_alert_type" property of a PublisherTypeDefault object.

####Alert States

* **Inactive** - The alert is not processing. It will not begin processing until started (see alertManager.enableAlert).
* **Starting** - The alert is attempting to transition to the "Active" state. If the transition succeeds, the state will become "Active;" otherwise the state will revert to "Inactive."
* **Active** - The alert is processing; however, its conditions have not yet been met. The alert will stay in the "Active" state until the user stops it (see alertManager.disableAlert) or until the conditions are met.
* **Stopping** - The user has requested that alert processing stop. Once this operation is complete, the alert will return to the "Inactive" state.
* **Triggered** - The alert's conditions have been met. The alert can be manually restarted (see alertManager.enableAlert).


##AlertManager Operations

###getServerVersion

Returns the version of the server in a JSON object, as follows:

JSON-in:

	{
	}

JSON-out:

	{
	  "semver": "0.0.1"
	}


###getTargets

Returns an array of the supported "Target" objects.

JSON-in:

	{
	}

JSON-out:

	An array of "Target" objects


###getProperties

Returns an array of supported "Property" objects.

JSON-in:

	{
	}

JSON-out:

	An array of "property" objects


###getOperators

Returns an array of supported "Operator" objects.

JSON-in:

	{
	}

JSON-out:

	An array of "Operator" objects


###getPublisherTypes

Returns an array of supported "PublisherType" objects.

JSON-in:

	{
	}

JSON-out:

	An array of "PublisherType" objects


###getPublisherTypeDefaults

Returns an array of "PublisherTypeDefault" objects.

JSON-in:

	{
	    "user_id": "barchart-test-user",
	    "alert_system": "barchart.com"
	}

JSON-out:

	An array of "PublisherTypeDefault" objects


###assignPublisherTypeDefault

Updates a PublisherTypeDefault for a specific user.

JSON-in:

	{
		"publisher_type_id": 1,
		"user_id": "barchart-test-user",
		"alert_system": "barchart.com",
		"default_recipient": "123-456-7890",
		"active_alert_types": [
			"price",
			"news"
		],
		"allow_window_start": "06:00",
		"allow_window_end": "20:30",
		"allow_window_timezone": "America/Chicago"
	}

JSON-out:

	An "PublisherTypeDefault" object.

Field details:

* active_alert_types - An array of strings. If the alert's type matches
a value contained within this array, then alert will be published using
the publisher type. Currently valid strings are: ("news" and "price").
* allow_window_start - When an alert is triggered, if the current time
is after the "allow_window_start" time and before the "allow_window_end"
time, then the publisher will be used. Otherwise the triggered alert
will not be published (using this publisher type). No restriction exists
if the value of this property is null.
* allow_window_end - When an alert is triggered, if the current time
is after the "allow_window_start" time and before the "allow_window_end"
time, then the publisher will be used. Otherwise the triggered alert
will not be published (using this publisher type). No restriction exists
if the value of this property is null.
* allow_window_timezone - The timezones that qualify the "allow_window_start"
and "allow_window_end" times. Use the Barchart.Alerts.timezone.getTimezones
function to get a list of valid timezone strings.


###createAlert

The following JSON object can be used to create an alert. The input is a
simplified version of the "Alert" object.

JSON-in:

	{
	    "name": "My Test Alert",
	    "user_id": "barchart-test-user",
	    "alert_system": "barchart.com",
	    "automatic_reset": false,
	    "alert_type": "price",
	    "user_notes": "This alert was created for fun and profit."
	    "conditions": [ {
	        "property": {
	            "property_id": 1,
	            "target":{
	                "identifier": "AAPL"
                }
            },
            "operator":{
                "operator_id": 3,
                "operand": "99"
            }
        } ],
        "publishers":[ {
            "type": {
                "publisher_type_id": 1
            },
            "use_default_recipient": false,
            "recipient": "123-456-7890",
            "format": "Apple stock is falling"
        }, {
		   "type": {
			   "publisher_type_id": 1
		   },
		   "use_default_recipient": true,
		   "format": "Apple stock is falling"
	   } ]
    }

JSON-out:

	An "Alert" object.


###retrieveAlert

Gets a single "Alert" object, using its identifier:

JSON-in:

	{
	    "alert_id": "59ddfd0b-89db-4886-85c3-eb8e7a289390"
	}

JSON-out:

	An "Alert" object.


###editAlert

A composite operation that deletes an existing alert and then recreates it.

JSON-in:

	An "Alert" object.

JSON-out:

	An "Alert" object.


###enableAlert

Causes an "Alert" object to begin testing for matched conditions.
Notifications will be sent as soon as the alert conditions are matched.

JSON-in:

	{
	    "alert_id": "59ddfd0b-89db-4886-85c3-eb8e7a289390"
	}

JSON-out:

	The "Alert" object that was enabled.


###disableAlert

Stops an "Alert" object from processing. No notifications will be sent.

JSON-in:

	{
	    "alert_id": "59ddfd0b-89db-4886-85c3-eb8e7a289390"
	}

JSON-out:

	The "Alert" object that was disabled.


###deleteAlert

Deletes a single "Alert" object, using its identifier:

JSON-in:

	{
	    "alert_id": "59ddfd0b-89db-4886-85c3-eb8e7a289390"
	}

JSON-out:

	The "Alert" object that was deleted.


###retrieveAlerts

Retrieves all the "Alert" objects for a user account:

JSON-in (example 1, required properties):

	{
	    "user_id": "barchart-test-user",
	    "alert_system": "barchart.com"
	}

JSON-in (example 2, optional filter, only "price" alerts):

	{
		"user_id": "barchart-test-user",
		"alert_system": "barchart.com",
		"filter": {
			"alert_type": "price"
		}
	}
	
JSON-in (example 3, optional filter, only alerts refer to AAPL):

	{
		"user_id": "barchart-test-user",
		"alert_system": "barchart.com",
		"filter": {
			"symbol": "AAPL"
		}
	}
	
JSON-in (example 4, optional filter, only alerts where the target is AAPL):

	{
		"user_id": "barchart-test-user",
		"alert_system": "barchart.com",
		"filter": {
			"target": {
				"identifier": "AAPL"
			}
		}
	}
	
JSON-in (example 5, optional filter, only alerts where a condition refers to AAPL):

	{
		"user_id": "barchart-test-user",
		"alert_system": "barchart.com",
		"filter": {
			"condition": {
				"operand": "AAPL"
			}
		}
	}

JSON-out:

	An array of "Alert" objects belonging to the specified user.


##Utility Functions

###Barchart.Alerts.timezone.getTimezones

Returns an array of strings describe valid timezones.


###Barchart.Alerts.timezone.guessTimezone

Returns a string that represents a guess of the user's current
timezone. A null value is returned if a guess cannot be made.


##Unit Testing

Gulp and Jasmine are used. Execute unit tests, as follows:

	gulp test