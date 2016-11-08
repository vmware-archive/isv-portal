# How to Build a Nozzle

## Overview

Cloud Foundry's logging system, Loggregator,
has a feature called firehose. The firehose includes the combined stream of logs 
and metrics from every apps as well as
the Cloud Foundry platform itself. Building a nozzle could be a solution for

* Draining metrics to an external dashboard product for sytem operators
* Sending every http request's details into a search tool
* Draining all application logs to an external system 
* Thinking a little differently, [this video shows auto-scaling an app based on firehose metrics](https://youtu.be/skJKvQfpKD4?t=1021)
  
[Firehose-to-syslog is a real world, production example](https://github.com/cloudfoundry-community/firehose-to-syslog)
of a nozzle.

## Building

Developing a nozzle should be done in Go, as this allows leveraging the
[NOAA library](https://github.com/cloudfoundry/noaa).
NOAA does the heavy lifting of establishing
an authenticated websocket connection to the logging system
as well as de-serializing the protocol buffers.

Draining the logs consists of:

1. Authenticating
1. Establishing a connection to the logging system
1. Forwarding events on to their ultimate destination

Authenticate by fetching a token from [uaa](https://github.com/cloudfoundry/uaa)
with a client having the `doppler.firehose` scope:

```go
	uaaClient, err := uaago.NewClient(uaaUrl)
	if err != nil {
		panic(err)
	}

	token, err := uaaClient.GetAuthToken(username, password, skipSSL)
	if err != nil {
		panic(err)
	}
```

Using the token, create a consumer and connect to the Firehose with a subscription id.
The id is important, since the firehose looks for connections having the same id and only
sends an event to one of those connections. This is how a nozzle developer can prevent
message loss during upgrades an other deployments: run at least two instances.

```go
	consumer := consumer.New(config.TrafficControllerURL, &tls.Config{
		InsecureSkipVerify: config.SkipSSL,
	}, nil)
	events, errors := consumer.Firehose(firehoseSubscriptionID, token)
```

`Firehose` will give back two channels: one for events and a second for errors.

The events channel receives six different types of events.

* ValueMetric: some platform metric at a point in time, emitted by platform components (for example, how many 2xx responses the router has sent out)
* CounterEvent: an incrementing counter, emitted by platform components (for example, a diego cells remaining memory capacity)
* Error: an error
* HttpStartStop: http request details, including both application and platform requests
* LogMessage:  a log message for an individual app
* ContainerMetric: application container information (for example, memory used)

For the full details on events, check the
[dropsonde protocol](https://github.com/cloudfoundry/dropsonde-protocol/tree/master/events).

The above events show how this data targets two different personae:
platform operators and application developers. Keep this in mind when designing an integration.

Having `doppler.firehose` scope gets a nozzle data for *every* application as well as the platform. 
Any filtering based on the event payload is the nozzle implementor's responsibility.
An advanced integration could do something like combine a
[service broker](service-brokers.md) with a nozzle to:

* Let application developers opt-in to logging (implementing filtering in the nozzle)
* Establish [SSO](https://docs.cloudfoundry.org/services/dashboard-sso.html) exchange for authentication such that developers only can access logs for their space's apps

For a full working example, see [firehose-nozzle](https://github.com/cf-platform-eng/firehose-nozzle).

## Deployment

Once you've build a nozzle, there are a couple options for deployment

#### As a Managed Service
Visit [managed service](managed-services.md)
for more details on what it means to be a managed service.

See also this
[example nozzle BOSH release](https://github.com/cloudfoundry-incubator/example-nozzle-release).

#### As an App

You could also deploy the nozzle on the elastic runtime itself.
Visit the Tile Generator's
[section on pushed appliactions](tile-generator.md#pushed-applications)
for more details.

## Open Source Nozzles

There are several open source examples you could use
as a reference for building your nozzle

[firehose-nozzle](https://github.com/cf-platform-eng/firehose-nozzle)

  * Example that simply writes to standard out
  * Useful starting point: scaffolding, tests, etc are in place

[example-nozzle](https://github.com/cloudfoundry-incubator/example-nozzle)

  * A single file implementation with no tests: as minimal as things can get


[gcp-tools-release](https://github.com/cloudfoundry-community/gcp-tools-release)

  * In addition to Nozzle data, it drains component syslogs and health data
  * Shows how to do a bosh-addon (for additional data outside a nozzle)
  * Nozzle is managed via bosh
  * Raw logs and metrics data take different paths in the source

[firehose-to-syslog](https://github.com/cloudfoundry-community/firehose-to-syslog)

  * Includes implementation code that adds additional metadata (potentially needed for acl)
      - Application name
      - Space guid & name
      - Org guid & name
  * [logsearch-for-cloudfoundry](https://github.com/cloudfoundry-community/logsearch-for-cloudfoundry) packages this nozzle as a BOSH release

[datadog-firehose-nozzle](https://github.com/cloudfoundry-incubator/datadog-firehose-nozzle)

  * A simpler implementation than other real examples

## Other References

* CF Summit Video [Monitoring Cloud Foundry: Learning about the Firehose](https://youtu.be/skJKvQfpKD4)
* [Loggregator github repo](https://github.com/cloudfoundry/loggregator/)
* [Overview of the Loggregator System](https://docs.cloudfoundry.org/loggregator/architecture.html)
* [Loggregator's Slack Channel](https://cloudfoundry.slack.com/messages/loggregator/)
