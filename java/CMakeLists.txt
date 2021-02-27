find_package(Java REQUIRED)
include(UseJava)

if (NOT DEFINED $ENV{JAVA_HOME_NATIVE})
  set (JAVA_HOME_NATIVE $ENV{JAVA_HOME})
  set (JAVAC $ENV{JAVA_HOME}/bin/javac)
  set (JAR $ENV{JAVA_HOME}/bin/jar)
else ()
  set (JAVAC $ENV{JAVA_HOME_NATIVE}/bin/javac)
  set (JAR $ENV{JAVA_HOME_NATIVE}/bin/jar)
endif ()

#[[
# Check that we can use javah
if(NOT Java_JAVAH_EXECUTABLE)
    message(FATAL_ERROR "Cannot locate javah executable.")
endif(NOT Java_JAVAH_EXECUTABLE)
#]]

set(JNI_HEADER_PATH "${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/tinybjar.dir/jni")
# message(STATUS "Generated jni headers in ${JNI_HEADER_PATH}")
set(CMAKE_JNI_TARGET TRUE)
file(GLOB JAVA_SOURCES "*.java")
add_jar(tinybjar ${JAVA_SOURCES}
                  MANIFEST ${CMAKE_CURRENT_BINARY_DIR}/manifest.txt
                  GENERATE_NATIVE_HEADERS tinyb-jniheaders DESTINATION "${JNI_HEADER_PATH}"
                  OUTPUT_NAME tinyb-${tinyb_VERSION_FULL}
)

set(JAVA_CLASSES tinyb.BluetoothAdapter
                 tinyb.BluetoothCallback
                 tinyb.BluetoothDevice
                 tinyb.BluetoothEvent
                 tinyb.BluetoothGattCharacteristic
                 tinyb.BluetoothGattService
                 tinyb.BluetoothGattDescriptor
                 tinyb.BluetoothManager
                 tinyb.BluetoothObject
                 tinyb.BluetoothType
                 tinyb.BluetoothException
                 tinyb.ObjectArgCallback
                 tinyb.ObjectArrayArgCallback)

#[[
add_custom_command (TARGET tinybjar
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E echo "Generating JNI headers.."
  WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/tinybjar.dir"
  COMMAND ${Java_JAVAH_EXECUTABLE} -d jni/ -jni ${JAVA_CLASSES}
)
#]] 

# set(JNI_HEADER_PATH "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/tinybjar.dir/jni")
# install (FILES $<TARGET_PROPERTY:tinybjar,JAR_FILE> DESTINATION ${CMAKE_INSTALL_LIBDIR}/../lib/java)

add_custom_target(
  upload_jars 
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMAND 
      mvn 
        deploy:deploy-file 
        -DgroupId=com.gpaglia.bluetooth 
        -DartifactId=tinyb 
        -Dversion=${tinyb_VERSION_FULL} 
        -Dpackaging=jar 
        -DrepositoryId=LIPPFH0T -Durl=http://LAPTOP-LIPPFH0T:1080/repository
        # -Dfile=${CMAKE_CURRENT_BINARY_DIR}/blescan-api-${VERSION}.jar
        -Dfile="$<TARGET_PROPERTY:tinybjar,JAR_FILE>"
    COMMAND 
      ${CMAKE_COMMAND} 
        -E echo "Uploaded blescan-api, blescan-hci version ${VERSION} to repository"
)

add_subdirectory (jni)
