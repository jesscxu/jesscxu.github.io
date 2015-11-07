---
author: Jessica Xu
layout: post-full
title: Real Time Line Graph
featimg: realtime_line_preview.png
type: visualization
tags: [d3, visualization] 
category: [visualization]
vizlink_small: realtime_line.html
---

Using the data from a light sensor built using an Arduino, I created a graph that would update data in a real time visualization using the Epoch library.





<br>


{% highlight python %}
"""
Using an Arduino, RGB sensor, and Bluetooth device, I collected data in a readable format and then parsed through it to put it into a JSON file that could be used to make a data visualization. 
"""
import time
import datetime
import simplejson as json
import re
import string
import serial
import pymongo
from pymongo import MongoClient


class BluetoothSensorData:
    exclude = set(string.punctuation)
    exclude.remove('.')
    client = MongoClient()
    db = client.database
    collection = db.collection
    mongo_data = db.mongo_data

    def __init__(self, bluetooth_device, sensor_type, num_data_points):
        self.bluetooth_device = bluetooth_device
        self.sensor_type = sensor_type
        self.num_data_points = num_data_points



    def main(self):
        BluetoothSensorData.mongo_data.drop()

        if self.sensor_type == 'pressure':
            max_len = 155
            min_len = 140
        if self.sensor_type == 'RGB':
            max_len = 65
            min_len = 45
        count = 0
            
        while True and count != self.num_data_points:
            ser = serial.Serial(self.bluetooth_device, 9600, timeout=2)
            
            while True and count < self.num_data_points:
                print(count)
                try:
                    reading = ser.readline()
                    print(reading)
                    if reading is None:
                        raise serial.serialutil.SerialException
                    elif not (len(reading) > max_len or len(reading) < min_len):
                        parts = reading.split(',')
                        data_labels = []
                        data_values = []
                        data_units = []
                        for i in range(len(parts)):
                            data_labels.append(parts[i].split(':')[0])
                            data_values.append(parts[i].split(':')[1].split(' ')[0])
                            data_units.append(parts[i].split(':')[1].split(' ')[1])
                        BluetoothSensorData.mongo_data.insert_one({
                            'sensor type': self.sensor_type, 
                            'data': data_labels,
                            'values': data_values,
                            'units': data_units,
                            'time': int(time.time()),
                            'status': 'ok',
                            'device': self.bluetooth_device
                        })
                            
                        count += 1
     
                except serial.serialutil.SerialException:
                    print json.dumps({
                        'sensor type': self.sensor_type, 
                        'data': None,
                        'values': None,
                        'units': None,
                        'time': datetime.datetime.now().isoformat(),
                        'status': 'offline',
                        'device': self.bluetooth_device
                    })
        return



    def export_json_data_category(self):
        arduino_data = []
        for i in range(len(BluetoothSensorData.mongo_data.find()[0]['data'])):
            values = []
            for data_dict in BluetoothSensorData.mongo_data.find():
                values.append(float(data_dict['values'][i]))
            data = {}
            data[str(data_dict['data'][i])] = values
            arduino_data.append(data)

        with open (self.sensor_type + '_arduino_data_categories.json', 'w') as outfile:
            json.dump(arduino_data, outfile, indent=4)

        return arduino_data


{% endhighlight %}

<br>

Link to formatted JSON data: [real time line graph data](/json/realtime_line.json)




