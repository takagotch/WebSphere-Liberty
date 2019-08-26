### websphere-liberty
---
https://developer.ibm.com/wasdev/

```
USER root
RUN mkdir -p /myFolder && chown -R 1001:0 /myFolder
USER 1001

USER root
RUN chown 1001:0 /config/jvm.options
RUN chown 1001:0 /output/resources/security/ltpa.keys
USER 1001

FROM websphere-liberty:kernel
COPY --chown=1001:0 Sample1.war /config/dropins/
COPY --chown=1001:0 server.xml /config/
RUN installUtility install --acceptLicense defaultServer

FROM webspere-liberty:springBoot2
COPY --chown=1001:0 hellospringboot.jar /config/dropins/spring/

FROM websphere-liberty:springBoot2 as staging
COPY --chown=100.0 hellospringboot.jar /staging/myFatApp.jar
RUN springBootUtility thin \
  --sourceAppPath=/staging/myFatApp.jar \
  --targetThinAppPath=/staging/myThinApp.jar \
  --targetLibCachePath=\staging/lib.index.cache
FROM websphere-liberty:springBoot2
COPY --from=staging /staging/lib.index.cache /lig.index.cache
COPY --from=staging /staging/myThinApp.jar /config/dropins/spring/myThinApp.jar

FROM websphere-liberty:webProfile8
RUN apt-get update \
  && apt-get install -y language-pack-pt-base \
  && rm -rf /var/lib/apt/lists/*
ENV LANG pt_BR.UTF-8
```

```sg
docker run -d -p 80:9080 -p 443:9443 \ 
  -v /tmp/DefaultServletEngine/dropins/Sample1.war:/config/dropins/Sample1.war \
  websphere-libery:webProfile8
  
docker run -d -p 80:9080 \
  -v /tmp/DefaultServletEngine:/config \
  websphere-liberty:webProfile8
  
```

```
```


