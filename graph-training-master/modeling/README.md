# Modeling
- Download and start the TigerGraph docker image according to the instructions here: <https://hubconnect.uhg.com/docs/DOC-168569> (However, when starting the container, also expose port 9000 like so:)
```
docker run -i -t --name tigergraph -p 4142:14240 -p 9000:9000 tigergraph:latest
```
- After a few minutes, the TigerGraph GraphStudio interface should be visible at <http://localhost:4142>
- If you're running TigerGraph on another machine or with different ports, create a gradle-local.properties file in this directory with any properties from gradle.properties that you need to change, like so:
```
gsqlHost=myservername:14240
gsqlRest=myservername:9000
gsqlUsername=user
gsqlPassword=pass
```
- For older versions of tigergraph, add the tgVersion property in gradle.properties or a newly created gradle-local.properties to the appropriate version, like so: (versions 2.3.2+ supported)
```
tgVersion=2.3.2
```
- View available gradle tasks for loading data by running
```
./gradlew tasks
```
- Load some example data by running
```
./gradlew loadExercise4
```
- Start a gsql session with the server by running
```
./gradlew --console=plain gsqlShell
```
