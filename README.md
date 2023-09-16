import csv
import requests
import socks
import socket

# Function to check if an account is valid using a SOCKS proxy
def check_account_with_proxy(username, password, proxy_address, proxy_port, proxy_type):
    # Configure the SOCKS proxy
    socks_proxy = None
    if proxy_type == "socks4":
        socks_proxy = socks.SOCKS4
    elif proxy_type == "socks5":
        socks_proxy = socks.SOCKS5
    else:
        return "Invalid Proxy Type"

    # Create a SOCKS proxy connection
    socks.setdefaultproxy(socks_proxy, proxy_address, int(proxy_port))
    socket.socket = socks.socksocket

    # HTTP request through the proxy
    try:
        response = requests.get("https://example.com")  # Replace with your verification URL
        if response.status_code == 200:
            return "Valid Account"
        else:
            return "Invalid Account"
    except requests.exceptions.RequestException:
        return "Failed to connect through proxy"

# Load a list of accounts and proxies from a CSV file (replace with your data source)
def load_accounts_and_proxies(file_path):
    accounts_and_proxies = []
    with open(file_path, 'r') as file:
        reader = csv.reader(file)
        for row in reader:
            if len(row) == 5:  # Username, Password, Proxy Type, Proxy Address, Proxy Port
                accounts_and_proxies.append((row[0], row[1], row[2], row[3], row[4]))
    return accounts_and_proxies

# Main function to check accounts with proxies and write results to text files
def main():
    # Load accounts and proxies from a CSV file
    accounts_and_proxies = load_accounts_and_proxies("accounts_and_proxies.csv")

    # Create text files for results
    with open("account_status.txt", "w") as status_file, open("account_info.txt", "w") as info_file:
        # Check each account with the specified proxy
        for username, password, proxy_type, proxy_address, proxy_port in accounts_and_proxies:
            result = check_account_with_proxy(username, password, proxy_address, proxy_port, proxy_type)
            account_info = f"Username: {username}, Password: {password}, Result: {result}\n"
            info_file.write(account_info)

            # Determine if the account is paid or unpaid based on the result (customize this logic)
            if "Valid" in result:
                status = "paid"
            else:
                status = "unpaid"
            status_file.write(f"{username}:{status}\n")

if __name__ == "__main__":
    main()
