🚀 Graylog 3-Node HA Installation Guide

➡️ VIEW THE LIVE GUIDE

Click Here to Open the Interactive HTML Guide

(Note: You must enable GitHub Pages in your repository settings for this link to work! See "How to Use" below.)

This repository contains a clean, easy-to-read HTML version of a comprehensive guide for setting up a 3-Node Graylog High Availability (HA) cluster.

This guide is specifically tailored for a production environment where each server has 32GB of RAM and all three components (MongoDB, Graylog Data Node, and Graylog Server) are co-located on the same machines.

✨ Features

✨ Clean & Readable: A professional, responsive HTML layout for easy reading on any device.

📋 Copy-to-Clipboard: All code blocks include a "Copy" button for quick and error-free execution.

🔢 Step-by-Step: The guide is broken down into 5 logical stages, from system prep to final testing.

🔒 Anonymized: All specific IP addresses and hostnames have been replaced with placeholders (e.g., <NODE1_IP>) for security and general use.

🚀 Prod-Optimized: Includes memory (heap) configurations for MongoDB (8GB), Data Node/OpenSearch (12GB), and Graylog Server (8GB) to run stably on 32GB RAM servers.

⚙️ How to Use (To View the Live Guide)

Push Files: Ensure both README.md and graylog_ha_guide.html are pushed to your GitHub repository.

Enable GitHub Pages:

In your GitHub repo, go to Settings.

In the "Code and automation" section on the left, click on Pages.

Under "Build and deployment", for the "Source", select Deploy from a branch.

Set the "Branch" to main (or master) and the folder to / (root).

Click Save.

Wait & Access: Wait about 1-2 minutes for GitHub to build and deploy your site.

Click the Link: Go back to the main page of your repo (the README.md) and click the "View the Live Guide" link at the top. You will need to replace <YOUR_USERNAME> in the link with your actual GitHub username.

🤝 Contributing

If you find any errors, or have suggestions for improvements (e.g., updates for new Graylog versions), feel free to fork this repository, make your changes, and submit a pull request.
