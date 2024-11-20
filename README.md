# RED HAT DEBEZIUM SENIOR SOFTWARE QUALITY ENGINEER CHALLENGE

This is my challenge solution for the Red Hat Debezium Senior Software Quality Engineer position.

## Challenge

The challenge description is described in the following sections:

## Part 1

- Clone Debezium [core repository](https://github.com/debezium/debezium)
- Follow the [documentation](https://github.com/debezium/debezium/blob/main/README.md) and build Debezium locally, skip integration tests in the process
- Follow the [documentation](https://github.com/debezium/debezium/blob/main/debezium-connector-postgres/README.md) and execute single integration test PostgresMoneyIT for debezium-connector-postgres module
- Follow the [documentation](https://github.com/debezium/debezium/blob/main/debezium-connector-oracle/README.md) and execute single integration test OracleRowIdDataTypeIT for debezium-connector-oracle module for LogMiner adapter. Analyze module’s [pom.xml](https://github.com/debezium/debezium/blob/main/debezium-connector-oracle/pom.xml) to identify which Maven profiles must be enabled for successful execution.
- Save logs from the builds and deliver them for examination.

### Part 2 (Optional)

- Check [Debezium Operator](https://github.com/debezium/debezium-operator) repository
- Run a local Openshit/Kubernetes cluster: choose an arbitrary solution for that (kind, microk8s, crc, …)
- Deploy [PostgreSQL example](https://github.com/debezium/debezium-operator/tree/main/examples/postgres)
- Grab Debezium Server pod log and deliver it for examination

## Overview

In order to make an easier read I have separated the content of the two parts. You can find the solution for the first part [in here](PART-1.md) and the solution for the second part [in here](PART-2.md).

Both contents are describes what are the steps i have followed, the commands run and explained. Using the different links from all over the different documents, you can navigate to read the different contents. The log files are stored in the `logs` folder and in the corresponding subfolder to each exercise.

In case something is not correctly explained and you have any doubts, please contact me.

## Notes

* In the second exercise I have faced a problem with the debezium-server pod. The pod would not start. In that exercise there is a note explaining the situation and how I would solve it. If I had more time for the challenge, I would have spent it on this. Therefore, this is one of the points I would work on if I had more time.

## Feedback

In the interview, it was mentioned that feedback is welcome, I wanted to add 3 points that I thought were interesting to share:

* Reading the documentation from the repository, there are quite a few commands ready to be copy-pasted. In those commands there is a `$` at the begining of each command. Removing it will make all the copy-paste and reading a bit more easier for everybody. [Example](https://github.com/debezium/debezium/blob/572675acb6c738b955bff991ffc298a708f44bae/README.md?plain=1#L106)
* In all the maven commands seen in the documentation the wrapper is not used. In all the sites they recommend to use the wrapper, mainly for two reasons. The first one, to remove a dependency and the second, that all people (developers, testers and users) will use the same version.  
