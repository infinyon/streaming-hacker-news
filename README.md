# streaming-hacker-news
Streaming hacker news rss feed using InfinyOn Cloud and Fluvio
```mermaid
graph LR;

feed("Hackers News RSS feed")
http("HTTP Connector(s)")
rss_json("Smartmodule RSS to JSON")
jolt("Smartmodule Jolt Shift Transform")
array-map("Smart module JSON array to stream of Records")
topic("Fluvio topic")

subgraph fc[Fluvio Cluster]

http -->rss_json-->jolt-->array-map--> topic
end
feed --> http

```

## Install Fluvio CLI

```
curl -fsS https://packages.fluvio.io/v1/install.sh | bash
```
```
echo 'export PATH="${HOME}/.fluvio/bin:${PATH}"' >> ~/.bashrc
```
```
source ~/.bashrc
```

## Login into Infinyon Cloud 

```
fluvio cloud login --use-oauth2
```

## List of running connectors 

```
fluvio cloud connector list
```

## List available smart modules 

```
fluvio hub list
```

## Download required SmartModules

Use `fluvio hub download` to download the smartmodules to your cluster:
```
fluvio hub download infinyon/jolt@0.1.0
fluvio hub download infinyon-labs/rss-json@0.1.0
fluvio hub download infinyon-labs/array-map-json@0.1.0
```

Use fluvio sm list to confirm that they have been downloaded:
```
fluvio sm list
```
## Create transformation
Create a transformation configuration file. A configuration file may invoke one or more smartmodules. In our case, we'll chain all three smartmodules. Note: jolt smartmodule takes a spec that expresses the transformation:
```
transforms:
  - uses: infinyon-labs/rss-json@0.1.0
  - uses: infinyon/jolt@0.1.0
    with:
      spec:
      - operation: shift
        spec:
          items: ""
  - uses: infinyon-labs/array-map-json@0.1.0
```

## (Optional) Test transformations locally 
To test transformations locally install SMDK with: 
```
fluvio install smdk
```
download sample data
```
curl -o hackersnews.xml https://hnrss.org/newest
```
Use smdk to test the transformation locally:
```
smdk test --transforms-file transformations.yml --file hackersnews.xml --raw | tail -n +2 | jq
```

Note: | tail -n +2 | jq is optional to make the output readable.

## Deploy HTTP connector with transformations into Infinyon cloud
```
fluvio cloud connector create  -c hackernews-connector.yaml
```

## Check the connector by consuming records from hackernews topics

```
fluvio consume hackernews -Bd
```
