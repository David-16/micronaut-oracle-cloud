FROM oracle/graalvm-ce:20.2.0-java11 as graalvm
RUN gu install native-image

WORKDIR /home/app
COPY build/layers/libs /home/app/libs
COPY build/layers/resources /home/app/resources
COPY build/layers/application.jar /home/app/application.jar

RUN native-image --static \
                 -H:Name=func \
                 --report-unsupported-elements-at-runtime \
                 --initialize-at-build-time=example \
                 --no-server \
                 -H:IncludeResources=simplelogger.properties \
                 --no-fallback \
                 -cp /home/app/libs/*.jar:/home/app/resources:/home/app/application.jar \
                 com.fnproject.fn.runtime.EntryPoint


# need socket library from Fn FDK
FROM fnproject/fn-java-fdk:1.0.105 as fnfdk

# Package native binary in minimal container image
FROM oraclelinux:7-slim
WORKDIR /function
RUN groupadd -g 1000 fn && useradd --uid 1000 -g fn fn
COPY --from=graalvm /home/app/func func
# get unix socket libraries from FDK container image
COPY --from=fnfdk /function/runtime/lib/* .
ENTRYPOINT ["./func", "-XX:MaximumHeapSizePercent=80"]
CMD [ "io.micronaut.oraclecloud.function.http.HttpFunction::handleRequest" ]
