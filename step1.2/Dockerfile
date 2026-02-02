FROM ubuntu:20.04

# Install dependencies (without jasper which is deprecated)
RUN apt-get update && apt-get install -y \
    openjdk-8-jdk \
    maven \
    libpng-dev \
    libdc1394-22 \
    libsm6 \
    libxext6 \
    libxrender-dev \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy the entire application directory
COPY . /app

# Install OpenCV jar to Maven repository
RUN mvn install:install-file \
    -Dfile=/app/lib/opencv-3410.jar \
    -DgroupId=org.opencv \
    -DartifactId=opencv \
    -Dversion=3.4.10 \
    -Dpackaging=jar

# Build the application
RUN mvn clean package -DskipTests

# Expose the port
EXPOSE 8080

# Set the entry command (use ubuntuupperthan18 libs for Ubuntu 20.04)
CMD ["java", "-Djava.library.path=lib/ubuntuupperthan18/", "-jar", "target/fatjar-0.0.1-SNAPSHOT.jar"]
