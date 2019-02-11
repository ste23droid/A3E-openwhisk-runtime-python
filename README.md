# A3E OpenWhisk Python Runtimes

### Build custom runtimes
To build the runtime containing **tensorflow-numpy-pillow**:
```
cd A3E-openwhisk-runtime-python
./gradlew core:a3e-alpine-tensorflow-numpy-pillow-python3:distDocker -PdockerImagePrefix=whisk
```

To build the runtime containing **opencv-numpy**:
```
cd A3E-openwhisk-runtime-python
./gradlew core:a3e-alpine-opencv-numpy-python2:distDocker -PdockerImagePrefix=whisk
```

To build the runtime containing **opencv-numpy-pillow**:
```
cd A3E-openwhisk-runtime-python
./gradlew core:a3e-alpine-opencv-numpy-pillow-python2:distDocker -PdockerImagePrefix=whisk
```

