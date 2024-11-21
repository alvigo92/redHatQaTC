# PART 1 

This is th README regarding the solution to the first part of the challenge for the Red Hat Senior Quality Engineer position. 

## Clone the repo

Clonning the repo is something pretty straigth straightforward. There are three different ways to perform it:

```bash
git clone https://github.com/debezium/debezium.git
```

```bash
gh repo clone debezium/debezium
```

```bash
git clone git@github.com:debezium/debezium.git
```

Any of the above solutions will work and will download the repository into your local machine. 
Once this step has been done, the next one is change the directory into the debezium repository with:

```bash
cd debezium
```

And the last step is to run the following command in order to download all the dependencies, if we run this first, the next steps in the challenge will be quite faster. In my case this took a bit more than a couple of hours.

```bash
mvn dependency:resolve
```

## Build debezium locally and skip the integration tests

Once we have all the dependencies downloaded (as mentioned above this can take quite a long time) the next step is to locally build debezium. In this specifc case it has been requested to do not run the integration tests. To achieve that we need to run it with the option `-DskipITs` but if we want to skip all the tests we should add the option `-Dquick` as well. 

In my case I have run the following command and also added the `tee` command to grab the output. 

```bash
mvn clean install -DskipITs | tee build.log
```

The build.log can be found in [this file](./logs/part1/build.log).

## PostgresMoneyIT for debezium-connector-postgres module

A new scenario comes in and in this case I have to run one specific integration test case (called `PostgresMoneyIT`) for a given debezium module (called `debezium-connector-postgres`). In a very similar way as the previous step, we are running the scneario with the following command:

```bash
cd debezium-connector-postgres; mvn clean verify -Dit.test=PostgresMoneyIT -DskipTest | tee PostgresMoneyIT.log
```

In this case, we are changing the directory to the mentioned module. Then we run only the mentioned integration test with `-Dit.test=PostgresMoneyIT` and grab the logs from. They are located in [this file](./logs/part1/PostgresMoneyIT.log). Additionally, to make the results faster, I have added the option `-DskipITs` and skipped all the unit test. 

## OracleRowIdDataTypeIT for debezium-connector-oracle module and LogMiner adapter

Stepping a bit deeper, this new scenario includes the option to run the command (more or less the same as above) but with a specific profile. In this case we are running the command with the profiles called `oracle-tests` and `oracle-docker` which will run the integrations tests for the connector and the logMiner adapter, creating the oracle container with the needed configuration. 

```bash
cd debezium-connector-oracle; mvn clean verify -Dit.test=OracleRowIdDataTypeIT -DskipTest -P oracle-tests,oracle-docker | tee OracleRowIdDataTypeIT.log
```

In the same way as previous scenarios, the logs can be found [here](./logs/part1/OracleRowIdDataTypeIT.log). In this case we change the directory to the specified module. Then we run the mentioned integration test case, skipping unit test to make it faster. And last option, the profile. This profile has been selected based on [this profile](https://github.com/debezium/debezium/blob/572675acb6c738b955bff991ffc298a708f44bae/debezium-connector-oracle/pom.xml#L360) defined in the module's pom given in the statement. 

### Note

As mentioned in previous emails, there is a problem downloading the image `quay.io/rh_integration/dbz-oracle:19.3.0`, that seems to be non public and therefore the test cannot complete with success. 
