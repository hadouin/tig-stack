## TODO:
- [ ] Faire la query gjson pour reformater la reponse de la requete instagram

![Logo](https://user-images.githubusercontent.com/64506580/159311466-f720a877-6c76-403a-904d-134addbd6a86.png)


# Telegraf, InfluxDB, Grafana (TIG) Stack

Gain the ability to analyze and monitor telemetry data by deploying the TIG stack within minutes using [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/).




## ⚡️ Getting Started

Clone the project

```bash
git clone https://github.com/huntabyte/tig-stack.git
```

Navigate to the project directory

```bash
cd tig-stack
```

Change the environment variables define in `.env` that are used to setup and deploy the stack
```bash
├── telegraf/
├── .env         <---
├── docker-compose.yml
├── entrypoint.sh
└── ...
```

Start the services
```bash
docker-compose up -d
```

## 💾 Add config
Customize the `telegraf.conf` file which will be mounted to the container as a persistent volume

```bash
├── telegraf/
│   ├── telegraf.conf <---
├── .env
├── docker-compose.yml
├── entrypoint.sh
└── ...
```

For an input of type JSON my solutions is to reformat with gjson to send it to influxdb automaticaly. 
- Copy a sample json response you get from api call then give it to chat gpt and specify wich field you want in your final object.
- Test the gjson query [here](https://gjson.dev/)
- tell which will be tags, the rest is fields (check for strings values on telegraf data format docs)

Example:
```toml
  # url of your call
  urls = [
    "https://graph.facebook.com/${FACEBOOK_PAGE_ID}/posts?fields=id,message,likes.summary(true),comments.summary(true)"
  ]
  # name of measurement
  name_override="facebook_stats"
  data_format = "json_v2"
  
  # Set the HTTP header if needed
  [inputs.http.headers]
    Authorization = "Bearer ${FACEBOOK_ACCESS_TOKEN}"
  
  [[inputs.http.json_v2]]
    [[inputs.http.json_v2.object]] # will get metrics for fields of each object of the array 
      # query to get array of objects with what you want
      path="data.#.{id:id,message:message,likes_total_count:likes.summary.total_count,comments_total_count:comments.summary.total_count}"
      # here i speciify only tags since i already got rid of what i didn't need (you could exclude things see doc)
      tags=["id","message"]
```
