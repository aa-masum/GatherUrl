#!/bin/bash
figlet "Gather URL"

# Prompt the user for the subdomain file
read -p "Enter the subdomain file (e.g., subdomain.txt): " subdomain_file

# Check if the file exists
if [[ ! -f "$subdomain_file" ]]; then
  echo "Error: File $subdomain_file not found!"
  exit 1
fi

# Gather URLs using waybackurls and save them to wayurl.txt
echo "Gathering URLs using waybackurls..."
cat "$subdomain_file" | waybackurls | tee wayurl.txt

# Gather URLs using katana and save them to katanaurl.txt
echo "Gathering URLs using katana..."
katana -list "$subdomain_file" | tee katanaurl.txt

# Gather URLs using gau with threads and save them to gauurl.txt
echo "Gathering URLs using gau..."
cat "$subdomain_file" | gau --threads 5 | tee gauurl.txt

#  Merge all the URLs into url.txt file and remove duplicate by uro
echo "Merging all URLs into url.txt..."
sudo cat wayurl.txt katanaurl.txt gauurl.txt | uro > url.txt

#  Check for alive URLs using httpx and filter out 404 and 403 status codes
echo "Checking for alive URLs (excluding 404, 403 status)..."
httpx -l url.txt -fc 404,403 -o aliveurl.txt
sudo rm katanaurl.txt gauurl.txt wayurl.txt url.txt
# Notify user that the script is done
echo "Alive URLs have been saved to aliveurl.txt"
