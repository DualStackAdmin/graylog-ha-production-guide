Graylog 3-Node HA Installation Guide (HTML Version)

This repository contains a clean, easy-to-read HTML version of a comprehensive guide for setting up a 3-Node Graylog High Availability (HA) cluster.

This guide is specifically tailored for a production environment where each server has 32GB of RAM and all three components (MongoDB, Graylog Data Node, and Graylog Server) are co-located on the same machines.

Features

Clean & Readable: A professional, responsive HTML layout for easy reading on any device.

Copy-to-Clipboard: All code blocks include a "Copy" button for quick and error-free execution.

Step-by-Step: The guide is broken down into 5 logical stages, from system prep to final testing.

Anonymized: All specific IP addresses and hostnames have been replaced with placeholders (e.g., <NODE1_IP>) for security and general use.

Prod-Optimized: Includes memory (heap) configurations for MongoDB (8GB), Data Node/OpenSearch (12GB), and Graylog Server (8GB) to run stably on 32GB RAM servers.

How to Use

Download or clone this repository.

Open the graylog_ha_guide.html file in any modern web browser.

Follow the steps in the guide, replacing the placeholders (like <NODE1_IP>, YOUR_PASSWORD_SECRET_VALUE, etc.) with your own environment's values.

Contributing

If you find any errors, or have suggestions for improvements (e.g., updates for new Graylog versions), feel free to fork this repository, make your changes, and submit a pull request.
