# MikroTik-WireGuard-DDNS-Reconnector

## Description

This script is designed for MikroTik RouterOS devices and serves the purpose of handling WireGuard VPN connections in scenarios where peer domains are dynamic DNS (DDNS) addresses. It ensures the stability of the VPN connection by monitoring and restarting WireGuard peers, synchronizing connections with dynamic domain names. Ideal for environments requiring DDNS handling, such as mobile devices, home networks, etc.

## Key Features

- Automatically detects and restarts WireGuard connections
- Handles scenarios where peer domains are dynamic DNS addresses
- Specifically designed for MikroTik RouterOS devices

## Usage

1. Add the following script content to your MikroTik RouterOS device.
2. Configure the dynamic DNS address for your WireGuard peer.
3. The script will automatically monitor and maintain the connection.

## Installation

- Copy the following script content and execute it in the terminal:

```routeros
/system/script/add name=wg-ddns source={
:foreach wg in=[/interface/wireguard/find disabled=no] do={
 :local peer [/interface/wireguard/get $wg name]
  :foreach i in=[/in/wireguard/peers/find interface=$peer endpoint-address~"[a-z]\$"] do={
    :local currentIP [/interface/wireguard/peers/get $i value-name=current-endpoint-addres]
    :local resolvedIP [:resolve [/interface/wireguard/peers/get $i value-name=endpoint-address]]
    :put "Checking $peer"
    :put "Current IP: $currentIP"
    :put "Resolved IP: $resolvedIP"
    :if ( $currentIP != $resolvedIP ) do={
      :put "IPs are not the same, restarting $peer"
      /in/wireguard/ disable $peer; :delay 1s; /in/wireguard/ enable  $peer;	
      };
    };
  };
}
/system/scheduler/add name=wg-ddns-scheduler interval=5m on-event=wg-ddns

