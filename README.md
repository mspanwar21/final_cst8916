
# Real-time-Monitoring-System-for-Rideau-Canal-Skateway

### **Submission By** 
 **Mohit Singh Panwar(041167761)**


## Scenario Description

To ensure citizen safety, the City of Ottawa has implemented a real-time data-gathering platform across three popular ice skating locations—Dow's Lake, Fifth Avenue, and the National Arts Centre (NAC). The platform collects critical metrics such as Ice Thickness, Surface Temperature, Snow Accumulation, and External Temperature. This data enables the city to make informed decisions regarding the opening or closing of skating rinks.

- Sensors are installed across the three locations to collect data continuously.
- Incoming data is processed in real time to support decision-making regarding rink availability.
- The collected data is stored in Azure Storage for future analysis and to aid in long-term planning and safety assessments.

## System Architecture

- **IoT Sensors:** Collect real-time data from the Rideau Canal, Dow's Lake, Fifth Avenue, and NAC.
- **Azure IoT Hub:** All sensor data is collected and centralized in the IoT Hub.
- **Azure Stream Analytics:** The data is analyzed and processed as required, such as calculating average ice thickness or snow accumulation on the surface.
- **Azure Blob Storage:** The processed data is stored in CSV/JSON formats for future analysis.

Data flow Diagram:

![Screenshot 2025-04-10 122425](https://github.com/user-attachments/assets/34555c6c-562c-485c-ae90-fec702160595)


## Implementation Details

### IoT Sensor Simulation

- **Description:** Simulated IoT sensors generate and send data to Azure IoT Hub every 10 seconds, including location, ice thickness, surface temperature, snow accumulation, external temperature, and timestamp.
- **JSON Payload Structure:**

```json
{
  "location": "Dow's Lake",
  "iceThickness": 27,
  "surfaceTemperature": -1,
  "snowAccumulation": 8,
  "externalTemperature": -4,
  "timestamp": "2024-11-23T12:00:00Z"
}
```

### 3.2 Azure IoT Hub Configuration

#### Steps to Set Up IoT Hub:
1. **Create an IoT Hub**:
   - Navigate to the Azure portal.
   - Create a new IoT Hub and select the **Standard** tier to enable advanced features like message routing.
2. **Add a Device**:
   - In the IoT Hub, go to the **Devices** section.
   - Add a new device by specifying a unique **Device ID**.
   - Copy the **Connection String** for the newly created device; it will be used in the simulation script for authentication and data transmission.

---

### 3.3 Azure Stream Analytics Job

#### Steps to Configure the Job:
1. **Create a Stream Analytics Job**:
   - In the Azure portal, search for **Stream Analytics Jobs** and click **Create**.
   - Provide a name, select a resource group, and choose **Cloud** as the hosting environment.
   - Click **Create** to initialize the job.

2. **Set Up Input**:
   - Navigate to the **Inputs** section and click **Add**.
   - Select **IoT Hub** as the input source.
   - Configure the input with:
     - Existing IoT Hub namespace.
     - Authorization policy: Use `iothubowner`.
     - Consumer group: Choose `$Default` or create a new one.
     - Data format: Set as **JSON**.

3. **Set Up Output**:
   - Go to the **Outputs** section and click **Add**.
   - Choose **Blob Storage** as the output destination.
   - Configure the output with:
     - Select an Azure Storage Account.
     - Specify or create a container for storing processed data.
     - (Optional) Add a folder structure for file organization.
     - Select the output format: **JSON** or **CSV**.

4. **Write the Query**:
   - Create a SQL-like query to process data from the input.
   - Example Query:
     ```sql
     SELECT
         IoTHub.ConnectionDeviceId AS DeviceId,
         AVG(iceThickness) AS AvgIceThickness,
         MAX(snowAccumulation) AS MaxSnowAccumulation,
         System.Timestamp AS EventTime
     INTO
         [output]
     FROM
         [input]
     GROUP BY
         IoTHub.ConnectionDeviceId, TumblingWindow(minute, 5)
     ```


# Sample Output

The processed data from Azure Stream Analytics is stored in JSON format. Examples of the aggregated output include:

```json
{"DeviceId":"sensor1","AvgIceThickness":28.233333333333334,"MaxSnowAccumulation":15.0,"EventTime":"2025-04-10T15:55:00.0000000Z"}
```

## Step 5: Save and Start the Job

1. Save the Stream Analytics configuration and the SQL query.
2. Click **Start** to activate the job and begin processing data in real time.

## Step 6: Verify the Job and Output

1. Navigate to the **Monitoring** tab in the Azure Portal.
2. Confirm that the job is running and processing incoming data correctly.
3. Access the configured Blob Storage container to check the processed data.

### Azure Blob Storage:

- **Data Storage Overview**
- **Storage Location**: Processed data is stored in `iotoutput`, the central storage for all Stream Analytics job outputs.
- **File Naming Convention**:
    - Example: `0_b879be9d239548248260ce7c319ef20a_1`.
- **File Format**: Data is saved in **JSON**, a structured and machine-readable format.
- **Benefits**: JSON allows easy parsing, querying, and integration with analytics tools and services.

#### Data Content:

Each JSON file contains aggregated data, including:

- **DeviceId**: sensor1 (e.g., dowslake, fifthavenue, nac).
- **AvgIceThickness**: Average ice thickness during the aggregation window.
- **MaxSnowAccumulation**: Maximum snow accumulation for the same window.
- **EventTime**: Timestamp marking the end of the aggregation period.

4. The script will simulate and display telemetry data for locations like Dow's Lake, Fifth Avenue, and NAC.

To stop the simulation, press `Ctrl + C`. The script will handle the disconnection gracefully.


# Usage Instructions

## Running the IoT Sensor Simulation:
- Install python from the official website

## Write Script:

Create a new file and save it with the `.py` extension.

```python
import random
import time
from datetime import datetime
from azure.iot.device import IoTHubDeviceClient, Message
import json

CONNECTION_STRING = "HostName=SkatewayIoTHub.azure-devices.net;DeviceId=sensor-simulator;SharedAccessKey=shRrDP+2e4tJXCI87zVSz9zz3CQxw6ql8i/K/YKW6g8="

LOCATIONS = ["Dow's Lake", "Fifth Avenue", "NAC"]

def generate_sensor_data():
    return {
        "location": random.choice(LOCATIONS),
        "iceThickness": random.randint(20, 35),
        "surfaceTemperature": round(random.uniform(-10, 0), 1),
        "snowAccumulation": random.randint(0, 15),
        "externalTemperature": round(random.uniform(-15, 5), 1),
        "timestamp": datetime.utcnow().isoformat() + "Z"
    }

def main():
    client = IoTHubDeviceClient.create_from_connection_string(CONNECTION_STRING)
    client.connect()
    print("Connected to IoT Hub")

    while True:
        data = generate_sensor_data()
        json_data = json.dumps(data)
        msg = Message(json_data)
        client.send_message(msg)
        print("Sent:", json_data)
        time.sleep(10)

if __name__ == "__main__":
    main()
```

This Python script simulates an IoT device sending telemetry data to an Azure IoT Hub:

1. It defines a `CONNECTION_STRING` to authenticate and connect the device (FifthAvenue) to the IoT Hub.
2. The `get_telemetry()` function generates random sensor data, including ice thickness, temperatures, and snow accumulation, along with a live timestamp.
3. In the `main()` function, the device connects to the IoT Hub and continuously sends telemetry data every 10 seconds using the `IoTHubDeviceClient`.
4. Each message is printed to the console for tracking, and the script stops gracefully when interrupted (e.g., with Ctrl+C).
5. The `if __name__ == "__main__":` block ensures the script runs when executed directly.

## Install Required Library:
Open the terminal and install the necessary library to simulate sensor data by running:

```bash
pip install azure-iot-device
```

## Update Connection String:
Replace the `CONNECTION_STRING` in the script with the connection string for your IoT device.

## Run the Script:
Execute the script in the terminal to start sending data to the IoT Hub.

## View Output:
The script will simulate and display telemetry data for locations like Dow's Lake, Fifth Avenue, and NAC.  
Each message will include details like:
- Ice thickness
- Surface temperature
- Snow accumulation
- External temperature

## Stop the Simulation:
Press `Ctrl + C` to stop the simulation. The script will handle this and disconnect gracefully.

---

# Configuring Azure Services:

## Resource Group
1. Create Resource group

![1](https://github.com/user-attachments/assets/0f3c92cb-e364-4596-9569-f0bfdc6cff97)


## Create an IoT Hub
1. In the Azure Portal, search for **IoT Hub** and click **Create**.
2. Provide a name for the IoT Hub and select a resource group.
3. Choose the **Free Tier** (if available) for testing purposes and create the IoT Hub.

![1 1](https://github.com/user-attachments/assets/b40ee493-5d3b-4c78-bf65-049deea50e2a)


## Stroage account

1. Create Stroage account and create container



![1 2](https://github.com/user-attachments/assets/6d0f9d41-ab63-4a40-bd2e-40d3aacbede0)




## Stream Analytics Job
1. In the Azure Portal, search for **Stream Analytics jobs** and click **Create**.
2. Provide a name for the job and select the appropriate resource group.
3. Choose **Cloud** as the hosting environment and create the job.

![1 3](https://github.com/user-attachments/assets/7048041a-03f2-446c-a3d7-fb0315a7aa21)



## Configure Input for Stream Analytics Job
1. In the Stream Analytics job, go to the **Inputs** section and click **Add**.
2. Choose **IoT Hub** as the input source.
3. Provide the following details:
   - **IoT Hub Namespace**: Select our IoT Hub.
   - **IoT Hub Policy Name**: Use the **iothubowner** policy.
   - **Consumer Group**: Use **$Default** or create a new consumer group in our IoT Hub.
   - **Serialization Format**: Choose **JSON**.

![2](https://github.com/user-attachments/assets/a0e4b436-8d5d-4801-b53a-f729ae1082c5)



## Write the Stream Analytics Query
1. Go to the Query tab and replace the default query with the following:

 ```sql
     SELECT
         IoTHub.ConnectionDeviceId AS DeviceId,
         AVG(iceThickness) AS AvgIceThickness,
         MAX(snowAccumulation) AS MaxSnowAccumulation,
         System.Timestamp AS EventTime
     INTO
         [output]
     FROM
         [input]
     GROUP BY
         IoTHub.ConnectionDeviceId, TumblingWindow(minute, 5)
 ```


**Explanation:** This query processes real-time data in Azure Stream Analytics. It calculates the average temperature and humidity from incoming data, grouped by the devices location, over 60-second intervals using a tumbling window. The results include the device ID, the average values, and the event timestamp. The processed data is then saved to a specified output location.

**Preview or Download Files**
Choose a file (e.g., a .json file) to view its contents directly in the Azure Portal.
Alternatively, download the file to your local machine for in-depth analysis using tools like a text editor, JSON viewer, or data processing software.

![4](https://github.com/user-attachments/assets/26981924-b254-4ccb-9be1-10348b7eef5c)
![4 1](https://github.com/user-attachments/assets/10acbb07-4f40-4a84-a292-6e0365b1fa9f)


# Results

## Key Findings

The Stream Analytics job successfully processed real-time data from IoT sensors and stored the aggregated results in Azure Blob Storage. The key metrics derived from this data include:

- **Average Ice Thickness**: Offers insights into ice conditions over specific time intervals.
- **Maximum Snow Accumulation**: Shows the highest snow accumulation during those intervals.

## Sample Aggregated Outputs

The processed data is saved in the `output.json` file.

## Accessing the Data

You can access the stored files by navigating to the `iotoutput` container in our Azure Blob Storage account.

# Reflection:
- Discuss any challenges faced during implementation and how they were addressed.

### Experience

The setup of the Stream Analytics job went smoothly without any problems. Each step, including configuring inputs, outputs, writing the query, and processing the data, was completed successfully.

### Learning

This project gave me useful experience in real-time data processing using Azure Stream Analytics. I learned how to integrate Azure services like IoT Hub and Blob Storage to build a complete data processing pipeline.
