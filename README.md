## ENet-SOCKS5

ENet-SOCKS5 is a library that extends the functionality of ENet by adding support for SOCKS5 proxies. It allows you to create and manage network connections over SOCKS5 proxies, enabling secure and efficient communication between your application and remote servers. While it has been tested on Linux devices, Windows compatibility has not been verified yet.

**Notice:** Please ensure that the provided proxy supports the UDP protocol!

### Build

```
cmake . && make
```

### Example

```cpp
struct ProxyInfo
{
  std::string ip;
  uint16_t port;
  std::string username;
  std::string password;
} proxy_info = {};

ENetHost *host = nullptr;
ENetPeer *peer = nullptr;

bool ENetClient::connect(const std::string &server_ip, uint16_t server_port) noexcept
{
    if (host != nullptr)
        disconnect();

    host = enet_host_create(0, 1, 2, 0, 0);
    host->usingNewPacket = true;
    if (host == nullptr)
        return false;

    // if port defined
    if (proxy_info.port > 0)
    {
        host->usingProxy = true;
        host->proxyInfo = {
            proxy_info.ip.data(),
            proxy_info.port,
            {proxy_info.username.data(),
             proxy_info.password.data()}};
    }

    host->checksum = enet_crc32;
    if (enet_host_compress_with_range_coder(host) < 0)
        return false;

    ENetAddress address;
    enet_address_set_host(&address, server_ip.c_str());
    address.port = server_port;

    peer = enet_host_connect(host, &address, 2, 0);
    if (peer == nullptr)
        return false;
    return true;
}

void ENetClient::disconnect() noexcept
{
    if (peer != nullptr)
    {
        enet_peer_disconnect(peer, 0);
        peer = nullptr;
    }
    if (host != nullptr)
    {
        enet_host_destroy(host);
        host = nullptr;
    }
}
```