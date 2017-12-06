## Basic Terms

* FETCH
  * To acquire data
  * Download
* WRITE
  * To write data
  * Upload
* TRAP
  * TODO: Under construction
* Storage
  * Server to accumulate data
* Gateway
  * Bridging the devices connected by IEEE1888 and different protocols
  * Watt meter <> RS485 <> GW <> IEEE1888 <> Storage
  * Temperature sensor <> I2C <> GW <> IEEE1888 <> Storage
* APP
  * Read and write data using IEEE1888
  * For example, if it is a viewer, the APP will FETCH and display from Storage
  * If it's analysis applications, APP does FETCH and writes analysis results from Storage
* Point, Point ID
  * Measurement points, similar to variables
    * Temperature of a certain point
    * Power at a certain point
  * Shown by URI. However, does not correspond to IP address. Only used as an unique identifier.
    * http://ujyu.net/laboratory/temperature
      * ujyu.net->Managed by Ujyu
      * laboratory->Location
      * temperature->Point
      * It is ok to use ujyu.net, however it will be better to possess your own unique domain.
      * Again, it is an identifier.
