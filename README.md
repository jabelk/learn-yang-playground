# Learning YANG Data Modeling By Playing in the NSO Playground 



# Overview

### What You’ll Learn
- Setting up NSO and the YANG models
- Looking at the person data model
- Looking at the router data model

### What You'll Need

- [Reserve the NSO Reservable Sandbox](https://devnetsandbox.cisco.com/RM/Diagram/Index/43964e62-a13c-4929-bde7-a2f68ad6b27c?diagramType=Topology)



# Setting Up NSO and the YANG Models

YANG is a core concept in network automation and is used in APIs like NETCONF and RESTCONF to define the data model. It can also be used to define other data models. Cisco Network Services Orchestrator (NSO) as a product was designed by folks who knew YANG very well and actually used it to define and customize NSO.

We can use Cisco NSO as a learning tool because you can customize the NSO application with YANG and view the results immediately. Even though this is not how most people will use YANG, seeing it in a new context may help you learn it in the more mainstream context of NETCONF and RESTCONF.

## Setting Up NSO

After your NSO Sandbox reservation is up, log in to the `10.10.20.49` system install server. It already has a populated device list and a running instance. Navigate to the NSO packages directory using the command `cd /var/opt/ncs/packages/`, and issue the `ls` command to view the existing packages NSO is using:

```bash
[developer@nso ~]$ cd /var/opt/ncs/packages/
[developer@nso packages]$ ls
cisco-asa-cli-6.12  cisco-iosxr-cli-7.32  resource-manager  svi_sample_service
cisco-ios-cli-6.67  cisco-nx-cli-5.20     selftest
[developer@nso packages]$
```

Now we are going to add a new package to NSO by using a built-in command to generate a skeleton set of files and folders that conform to the NSO package system. Issue the command `ncs-make-package --service-skeleton python belk-model` to create a new package, and then `tree belk-model` to view the files and folders:

```
[developer@nso packages]$ ncs-make-package --service-skeleton python belk-model
[developer@nso packages]$ tree belk-model/
belk-model/
├── package-meta-data.xml
├── python
│   └── belk_model
│       ├── __init__.py
│       └── main.py
├── README
├── src
│   ├── Makefile
│   └── yang
│       └── belk-model.yang
├── templates
└── test
    ├── internal
    │   ├── lux
    │   │   ├── Makefile
    │   │   └── service
    │   │       ├── dummy-device.xml
    │   │       ├── dummy-service.xml
    │   │       ├── Makefile
    │   │       ├── pyvm.xml
    │   │       └── run.lux
    │   └── Makefile
    └── Makefile

9 directories, 14 files
[developer@nso packages]
```

Let's set up a second example package as well with a more network focused example. Issue the command `ncs-make-package --service-skeleton python router-model` to create another package called `router-model`:

```bash
[developer@nso packages]$ ncs-make-package --service-skeleton python router-model
[developer@nso packages]$ ls
belk-model          cisco-iosxr-cli-7.32  router-model
cisco-asa-cli-6.12  cisco-nx-cli-5.20     selftest
cisco-ios-cli-6.67  resource-manager      svi_sample_service
[developer@nso packages]$
```

## Putting the YANG in Place

We will only need to edit one file in each of the packages for our example to work&mdash;the YANG file.

First, we will remove the existing placeholder file. Rather than try to tweak them, it is easier to fully replace them, using the `rm /var/opt/ncs/packages/belk-model/src/yang/belk-model.yang` and `rm /var/opt/ncs/packages/router-model/src/yang/router-model.yang` commands:

```
[developer@nso packages]$ rm /var/opt/ncs/packages/belk-model/src/yang/belk-model.yang
[developer@nso packages]$ rm /var/opt/ncs/packages/router-model/src/yang/router-model.yang
```

Now we can put in new copies. Here are the contents of each file:

belk-model.yang

```javascript
module belk-model {
  namespace "http://com/example/belkmodel";
  prefix belk-model;
  container person {
    leaf name { type string;}
    leaf age {type uint32;}
    leaf favorite-color { type string;}
}}
```

router-model.yang

```javascript
module router-model {
  namespace "http://com/example/routermodel";
  prefix router-model;
  import ietf-inet-types {
    prefix inet;
  }
container router {
  leaf name { type string;}
  leaf address { type inet:ipv4-address;}
  leaf operational-status {
    type enumeration {
          enum up;
          enum down;}
  }}}
```

You can use [VS Code Remote Explorer](https://code.visualstudio.com/docs/remote/ssh), or you can use `vim` or `nano` as well:

```bash
[developer@nso packages]$ nano /var/opt/ncs/packages/belk-model/src/yang/belk-model.yang
[developer@nso packages]$ nano /var/opt/ncs/packages/router-model/src/yang/router-model.yang
```

Now that the YANG files are updated, we need to compile the YANG so that NSO can load them into the application. Navigate to the first package with the command `cd /var/opt/ncs/packages/belk-model/src/` and issue a `make` command to compile the first YANG file. Next, issue the command `cd /var/opt/ncs/packages/router-model/src/` and issue a `make` command to compile the second package.

```bash
[developer@nso packages]$ cd /var/opt/ncs/packages/belk-model/src/
[developer@nso src]$ ls
Makefile  yang
[developer@nso src]$ make
mkdir -p ../load-dir
mkdir -p java/src//
/opt/ncs/current/bin/ncsc  `ls belk-model-ann.yang  > /dev/null 2>&1 && echo "-a belk-model-ann.yang"` \
              -c -o ../load-dir/belk-model.fxs yang/belk-model.yang
[developer@nso src]$ cd /var/opt/ncs/packages/router-model/src/
[developer@nso src]$ make
mkdir -p ../load-dir
mkdir -p java/src//
/opt/ncs/current/bin/ncsc  `ls router-model-ann.yang  > /dev/null 2>&1 && echo "-a router-model-ann.yang"` \
              -c -o ../load-dir/router-model.fxs yang/router-model.yang
[developer@nso src]$
```

Now you must log in to NSO to reload the packages and load in the new YANG files. Log in to NSO using the `ncs_cli` command, and reload the packages using the `packages reload force` command:

```
[developer@nso packages]$ ncs_cli

User developer last logged in 2022-12-15T14:02:55.457177-08:00, to nso, from 10.10.20.49 using rest-https
developer connected from 192.168.254.11 using ssh on nso
developer@ncs# packages reload force

>>> System upgrade is starting.
>>> Sessions in configure mode must exit to operational mode.
>>> No configuration changes can be performed until upgrade has completed.
>>> System upgrade has completed successfully.
reload-result {
    package belk-model
    result true
}
reload-result {
    package cisco-asa-cli-6.12
    result true
}
reload-result {
    package cisco-ios-cli-6.67
    result true
}
reload-result {
    package cisco-iosxr-cli-7.32
    result true
}
reload-result {
    package cisco-nx-cli-5.20
    result true
}
reload-result {
    package resource-manager
    result true
}
reload-result {
    package router-model
    result true
}
reload-result {
    package selftest
    result true
}
reload-result {
    package svi_verify_example
    result true
}
developer@ncs#

```



# Looking at the Person Data Model

What is a data model? A data model is a well-understood and agreed-upon method to describe "something." For example, consider this simple data model for a person.

Person:
- **Name:** A single value string that includes both their first and last name
- **Age:** An integer to represent the number of years old that they are
- **Favorite Color:** A single value string that is a color

Data models form the foundation for model-driven APIs. For these APIs, data models define the syntax, semantics, and constraints of working with the API. They define data attributes and answer questions, such as the following:

- What is the range of a valid VLAN ID?
- Can a VLAN name have spaces in it?
- Should the values be enumerated and only support "up" or "down" for an admin state?
- Should the value be a string or an integer?

If you remember, one of the models we loaded into NSO is a person data model. It uses the YANG syntax to describe the same things we just said above:

belk-model.yang

```javascript
module belk-model {
  namespace "http://com/example/belkmodel";
  prefix belk-model;
  container person {
    leaf name { type string;}
    leaf age {type uint32;}
    leaf favorite-color { type string;}
}}
```

The YANG syntax used are the following:

- `module`: This is the name we use if we want to import or reuse this in other places. It represents the top of the hierarchy.
- `container`: The container keyword is used to group items together, but not store any data itself. It just means that everything within the curly brackets under `container person` is all within one big umbrella. This will make more sense when we see the output.
- `leaf name`, `leaf age`, `leaf favorite-color` are single inputs (YANG leafs ask for a single value), and they give simple data types of string and integer.

YANG data models are hierarchical and use keywords to indicate the data types. The best way to learn them is to start using them and viewing the relationship between the data model definition and the payloads that it generates. The data model defines the constraints of the inputs and also forms the structure of the outputs.

## Seeing the YANG Model in Action

A data model needs an application to be used. The application reads in the YANG model and then takes in inputs to validate that they are the appropriate type per the data model (string versus integer versus other types). Then, the application uses the YANG data model to structure the output. In this case, NSO is the application, but generally, YANG is used in NETCONF or RESTCONF.

We already loaded the YANG model into NSO, so let's use NSO to generate some data and see the model in action.

Log in to NSO if you are not already in the prompt, using `ncs_cli`, and enter config mode with the command `conf`:

```
[developer@nso packages]$ ncs_cli

User developer last logged in 2022-12-15T14:02:55.457177-08:00, to nso, from 10.10.20.49 using rest-https
developer connected from 192.168.254.11 using ssh on nso
developer@ncs#
developer@ncs#
developer@ncs# conf
Entering configuration mode terminal
developer@ncs(config)#
```

Remember, the data model starts with the container of `person`, so type in that container name and hit `?` to see the available options:

```
developer@ncs(config)# person ?
Possible completions:
  age  favorite-color  name
```

Under `person`, we have three options&mdash;the three leafs we defined in our model. Choose `name` by typing in `name` after `person` and put in a value; you can use my name or your own. Press enter to save it into NSO:

```
developer@ncs(config)# person name Jason
```

Next, we want to save an `age`. Type in `age` and also type `?` to see the option. You can see that NSO loaded in the data type for us and we can only input an integer. If we try to save a string like `twelve`, it will fail. Type in an integer age, and then do the same process with `person favorite-color`:

```
developer@ncs(config)# person age ?
Possible completions:
  <unsignedInt>
developer@ncs(config)# person age 99
developer@ncs(config)# person favorite-color ?
Possible completions:
  <string>
developer@ncs(config)# person favorite-color green
```

Now that we have values stored in NSO for all the person attributes, we want to commit them to the application database using the `commit` command, and then exit the configuration dialogue using the `end` command:

```
developer@ncs(config)# commit
Commit complete.
developer@ncs(config)# end
```

Let's view the data stored by issuing a `show running-config person` command to see it in plaintext:

```
developer@ncs# show running-config person
person name    Jason
person age     99
person favorite-color green
developer@ncs#
```

Now let's see it with a structured data using the data model. Enter the command `show running-config person | display xml` to view the saved data in an XML format:

```xml
developer@ncs# show running-config person | display xml
<config xmlns="http://tail-f.com/ns/config/1.0">
  <person xmlns="http://com/example/belkmodel">
    <name>Jason</name>
    <age>99</age>
    <favorite-color>green</favorite-color>
  </person>
</config>
```

Let's see it with a JSON format using the command `show running-config person | display json`:

```json
developer@ncs# show running-config person | display json
{
  "data": {
    "belk-model:person": {
      "name": "Jason",
      "age": 99,
      "favorite-color": "green"
    }
  }
}
developer@ncs#
```

Compare the XML and JSON outputs to the original data model:

belk-model.yang

```javascript
module belk-model {
  namespace "http://com/example/belkmodel";
  prefix belk-model;
  container person {
    leaf name { type string;}
    leaf age {type uint32;}
    leaf favorite-color { type string;}
}}
```

Notice that the data model does not tell us what the values will be, but it gives us the flow and structure of how the output will look. This is incredibly helpful for automation because it gives you a predictable and programmatic way of expecting and consuming the data.



# Looking at the Router Data Model

Now let's see another example, but a bit more networking focused. The other data model we loaded in was a basic router data model.

Look at the `router-model`:

```javascript
module router-model {
  namespace "http://com/example/routermodel";
  prefix router-model;
  import ietf-inet-types {
    prefix inet;
  }
container router {
  leaf name { type string;}
  leaf address { type inet:ipv4-address;}
  leaf operational-status {
    type enumeration {
          enum up;
          enum down;}
  }}}
```

The main parts to focus on are:

- `module router-model`, which tells us the name of this data model. It is followed by a few lines about namespace and imports that we can ignore.
- `container router`: Just like before, the `container` YANG keyword is grouping all these leafs together into one bucket, but it is not actually storing any data itself.
- `leaf name` and `leaf address` are single inputs (YANG leafs ask for a single value), and they give simple data types of string for the name and an IP address for the address. YANG supports custom data types that use regular expressions behind the scenes to validate the input, so this data type will make sure the input follows the IPv4 address format A.B.C.D.
- `leaf operational-status` is another single input value, though the type of data that it is working with is a bit more complex. It uses an `enumeration` type to tell us that it can only be one of two possible values&mdash;either `up` or `down`.

There are a few other YANG keywords and data types, such as a YANG list, which can store a bunch of values looked up by a unique key (kind of like a Python dictionary), but the purpose of this tutorial is to get you general familiarity with YANG rather than an exhaustive tutorial of all the YANG syntax.

## Router Model in Action

Let's see this data model in action. Log in to the NSO CLI if you are not still in it, using the `ncs_cli` command in the Linux shell:

```
developer@ncs# exit
[developer@nso src]$ ncs_cli

User developer last logged in 2022-12-15T15:34:29.228187-08:00, to nso, from 192.168.254.11 using cli-ssh
developer connected from 192.168.254.11 using ssh on nso
developer@ncs#
```

Enter into configuration mode again using `conf` and type in the first data model level `router`. Hit `?` to see the options:

```
developer@ncs#
developer@ncs# conf
Entering configuration mode terminal
developer@ncs(config)# router
Possible completions:
  address  name  operational-status
developer@ncs(config)# router ?
Possible completions:
  address  name  operational-status
```

Note that the available options are defined per our data model above. Take a look at the `address` data type by typing in `router address ?` and then try to give an invalid IP address such as `router address 10.1.adlkfj`:

```
developer@ncs(config)# router address ?
Possible completions:
  <IPv4 address>
developer@ncs(config)# router address 10.1.adlkfj
--------------------------------------^
syntax error: "10.1.adlkfj" is not a valid value.
```

Now try again with a valid IP address like `router address 10.1.1.1` and then proceed to fill out the other values per the data model with values such as `router name sjc-gw1` and `router operational-status up`. Note that because we have the enumeration type for the operational status, there are only two options available:

```
developer@ncs(config)# router address 10.1.1.1
developer@ncs(config)# router name ?
Possible completions:
  <string>
developer@ncs(config)# router name sjc-gw1
developer@ncs(config)# router operational-status ?
Possible completions:
  down  up
developer@ncs(config)# router operational-status up
```

Save the changes to NSO for this set of inputs by typing in `commit` and then exit config mode with `end`:

```
developer@ncs(config)# commit
Commit complete.
developer@ncs(config)# end
```

View the data saved by typing in the `show running-config router` command, and then view it in the XML and JSON formats by typing in `show running-config router | display xml` and `show running-config router | display json`:

```
developer@ncs# show running-config router
router name        sjc-gw1
router address     10.1.1.1
router operational-status up
developer@ncs#
developer@ncs# show running-config router | display xml
<config xmlns="http://tail-f.com/ns/config/1.0">
  <router xmlns="http://com/example/routermodel">
    <name>sjc-gw1</name>
    <address>10.1.1.1</address>
    <operational-status>up</operational-status>
  </router>
</config>
developer@ncs# show running-config router | display json
{
  "data": {
    "router-model:router": {
      "name": "sjc-gw1",
      "address": "10.1.1.1",
      "operational-status": "up"
    }
  }
}
developer@ncs#
```

Now compare the XML and JSON payloads above to the original data model:

```javascript
module router-model {
  namespace "http://com/example/routermodel";
  prefix router-model;
  import ietf-inet-types {
    prefix inet;
  }
container router {
  leaf name { type string;}
  leaf address { type inet:ipv4-address;}
  leaf operational-status {
    type enumeration {
          enum up;
          enum down;}
  }}}
```

Note that the YANG model defined the structure of the inputs and outputs, but the application was the one that stored the actual data and then provided the payloads based on the YANG model. NETCONF and RESTCONF work the same way, where behind the scenes, they have YANG models defining the structure of your configuration and operational data and then use software to send back responses per the structure.



# Congratulations

## Learn More

- [Understanding Cisco Network Automation Essentials Course](https://ondemandelearning.cisco.com/apollo-alpha/mc_naec10_01/pages/1)
- [Understanding YANG Models](https://ondemandelearning.cisco.com/apollo-alpha/mc_naec10_08/pages/1)
- [DevNet Professional Paid Resources](https://learningnetworkstore.cisco.com/cisco-certified-devnet-professional)
- [DevNet Certifications Community](https://learningnetwork.cisco.com/s/topic/0TO3i0000008jY5GAI/devnet-certifications-community)
- [DevNet Webinars from Cisco Learning Network](https://learningnetwork.cisco.com/s/devnet-training-videos)

