# enet-socks5
ENet SOCKS5 support (single header file)

not tested on windows yet but working well on linux devices.

example (just a pseudo code, go implement it in your codebase)

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
    {
        disconnect();
    }
    host = enet_host_create(0, 1, 2, 0, 0);
    host->usingNewPacket = true;
    if (host == nullptr)
    {
        return false;
    }

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
