# IRC Server Project

**Author:** AbdulAzeez Shobjao (digitalpoolng@gmail.com)  
**License:** [MIT License](LICENSE)

---

## Overview

This project implements an IRC (Internet Relay Chat) server in C++98 that conforms to both [RFC 2812](https://www.rfc-editor.org/rfc/rfc2812.html) and modern IRC documentation ([Modern IRC Connection Registration](https://modern.ircdocs.horse/#connection-registration)). The server supports multiple clients using non-blocking I/O with a single `poll()` loop and provides the following core features:

- **Authentication & Registration:**
  - Clients must authenticate using the `PASS` command (if provided).
  - If included, `PASS` **MUST** precede the `NICK` and `USER` commands.
- **User Identification:**
  - Set a nickname with the `NICK` command.
  - Set a username with the `USER` command.
- **Channel Management:**
  - Clients can join channels using the `JOIN` command. Channel names must start with one of the following characters: `#`, `&`, `+`, or `!`.
  - When a channel is joined, the server sends a confirmation, the channel’s topic (if set), and a list of users (via numeric replies).
- **Private Messaging:**
  - Clients can send private messages using the `PRIVMSG` command.
  - If the `<msgtarget>` is a channel name, the message is broadcast to all channel members; otherwise, it is delivered to an individual client.
- **Channel Operator Commands:**
  - **KICK:** Eject a user from a channel.
  - **INVITE:** Invite a user to join a channel.
  - **TOPIC:** View or set the channel’s topic. If channel mode `t` is active, only operators can change it.
  - **MODE:** Adjust channel modes. Supported modes include:
    - `i`: Invite-only
    - `t`: Topic settable only by channel operators
    - `k`: Channel key (password)
    - `o`: Grant or remove channel operator privileges  
    (Additional modes as per RFC and modern extensions can be added as needed.)
- **Graceful Shutdown:**
  - The server installs a SIGINT handler so that when Ctrl+C is pressed, all client connections are closed, allocated memory is freed, and a closing message is printed in red.

---

## Table of Contents

1. [Features](#features)
2. [Build and Makefile](#build-and-makefile)
3. [Running the Server](#running-the-server)
4. [Usage and Commands](#usage-and-commands)
5. [Testing the Server](#testing-the-server)
6. [Persistent Buffer & Partial Data Handling](#persistent-buffer--partial-data-handling)
7. [Graceful Shutdown](#graceful-shutdown)
8. [Protocol References](#protocol-references)
9. [Contributing](#contributing)
10. [License](#license)

---

## Features

### Mandatory Features
- **Multi-Client Handling:**  
  Uses non-blocking I/O with a single `poll()` loop.
- **Authentication:**  
  Clients must send a `PASS` command (if provided) before sending `NICK` and `USER`.
- **Nickname & Username:**  
  Clients register using `NICK` and `USER` commands.
- **Channel Management:**  
  Clients can create or join channels using `JOIN`. On joining, they receive the channel topic (if set) and the list of channel members.
- **Private Messaging:**  
  Clients use `PRIVMSG` to send messages either privately or to channels.
- **Channel Operator Commands:**  
  Commands include `KICK`, `INVITE`, `TOPIC`, and `MODE` (with modes `i`, `t`, `k`, `o`).

### Additional Features
- **Graceful Shutdown:**  
  SIGINT (Ctrl+C) is handled to free resources and close connections gracefully.
- **Error Handling:**  
  Appropriate error messages are returned for invalid commands or conditions (e.g., wrong password, duplicate nicknames).

---

## Build and Makefile

Your Makefile must provide the following targets:
- **NAME** – Final executable name.
- **all**
- **clean**
- **fclean**
- **re**

### Build Instructions
1. **Clone the Repository:**
   ```bash
   git clone <repository-url>
   cd irc-server
   ```
2. **Build the Project:**
   ```bash
   make
   ```
3. **Clean Build Files:**
   ```bash
   make clean
   ```
4. **Full Clean (including binary):**
   ```bash
   make fclean
   ```
5. **Rebuild:**
   ```bash
   make re
   ```

---

## Running the Server

Run the server with:
```bash
./ircserv <port> <password>
```
- **`<port>`:** A listening port between 1024 and 65535.
- **`<password>`:** The connection password. If provided, the `PASS` command must be sent before `NICK` and `USER`.

**Example:**
```bash
./ircserv 6667 mypassword
```
The server will print a listening message and wait for client connections.

---

## Usage and Commands

### Client Commands
- **PASS `<password>`**: Authenticate with the server.
- **NICK `<nickname>`**: Set your nickname.
- **USER `<username>` 0 * :`<realname>`**: Set your username and real name.
- **JOIN `<channel>`**: Join or create a channel (channel name must start with `#`, `&`, `+`, or `!`).
- **PRIVMSG `<target>` :`<message>`**: Send a message to a channel or a private message to a user.
- **QUIT**: Disconnect from the server.

### Channel Operator Commands
- **KICK `<channel>` `<user>`**: Remove a user from a channel.
- **INVITE `<channel>` `<user>`**: Invite a user to join a channel.
- **TOPIC `<channel>` [ `<new topic>` ]**: View or change the channel topic.
- **MODE `<channel>` `<modes>` [ `<parameters>` ]**: Set channel modes (supported modes: `i`, `t`, `k`, `o`).

---

## Testing the Server

### 1. Using Command-Line Tools (nc / telnet)
- **Connect using netcat:**
  ```bash
  nc 127.0.0.1 6667
  ```
  - You will see a welcome message prompting for authentication.
  - Enter:
    ```
    PASS mypassword
    NICK MyNick
    USER MyUser
    ```
  - **Partial Data Test:**  
    Use `nc -C` (which sends CRLF) to simulate partial input. For instance, send parts of a command with ctrl‑D between segments; the persistent buffer will aggregate the input until a complete command (terminated by CRLF or LF) is received.

### 2. Using IRC Clients

#### **irssi (Terminal-based)**
- **Installation:**
  - Linux:
    ```bash
    sudo apt install irssi
    ```
  - macOS:
    ```bash
    brew install irssi
    ```
- **Usage:**
  1. Start irssi:
     ```
     irssi
     ```
  2. Add and connect to your server:
     ```
     /server add -auto -password mypassword localhost 6667
     /connect localhost
     ```
  3. Join a channel:
     ```
     /join #ch1
     ```
  4. Send messages:
     ```
     /msg #ch1 Hello, world!
     /msg OtherNick Hello in private!
     ```

#### **HexChat (Graphical)**
- **Installation:**
  - Linux:
    ```bash
    sudo apt install hexchat
    ```
  - macOS:
    ```bash
    brew install --cask hexchat
    ```
- **Usage:**
  1. Open HexChat.
  2. Create a new network (e.g., `MyServer`) and add your server details (`localhost/6667`).
  3. Connect to the server and join channels with `/join #ch1`.
  4. Use `/msg` to send private messages.

#### **weechat (Terminal-based)**
- **Installation:**
  ```bash
  sudo apt install weechat
  ```
- **Usage:**
  1. Start weechat:
     ```
     weechat
     ```
  2. Add and connect to your server:
     ```
     /server add myserver localhost/6667 -password mypassword
     /connect myserver
     ```
  3. Join a channel:
     ```
     /join #ch1
     ```

#### **Other Clients**
- **mIRC (Windows)** and **Kiwi IRC (Web-based)** are also supported. Follow their standard connection instructions.

### 3. Testing Channel & Operator Commands

- **JOIN:**  
  - Command: `JOIN #ch1`  
  - The server sends a JOIN confirmation, channel topic (if set), and a user list.
- **OPER:**  
  - Command: `OPER MyNick mypassword`  
  - Verifies operator privileges.
- **KICK:**  
  - As an operator, type: `KICK #ch1 UserX`  
  - The target user is removed and notified.
- **INVITE:**  
  - As an operator, type: `INVITE #ch1 UserY`  
  - The invited user receives an invitation message.
- **TOPIC:**  
  - To view: `TOPIC #ch1`  
  - To set: `TOPIC #ch1 New Topic Here`  
  - If channel mode `t` is set, only operators can change the topic.
- **MODE:**  
  - Examples:
    - `MODE #ch1 +i` (set invite-only)
    - `MODE #ch1 +k secretkey` (set channel key)
    - `MODE #ch1 +o OtherNick` (grant operator status)
    - `MODE #ch1 -i` / `-k` / `-o` (remove modes)

### 4. Testing Error Handling & Edge Cases

- Send incomplete or malformed commands (e.g., `PRIV` instead of `PRIVMSG`) and verify that appropriate error messages are returned.
- Test with multiple concurrent connections to ensure stability.

---

## Persistent Buffer & Partial Data Handling

The server uses a persistent buffer per client to accumulate incoming data until a complete command (terminated by CRLF or LF) is received. This ensures that if a command is sent in parts (for example, using ctrl‑D in `nc`), the server aggregates the data before processing the complete command.

---

## Graceful Shutdown

The server installs a SIGINT handler so that when you press Ctrl+C in the server’s terminal, it will:

- Close all client connections.
- Free all allocated memory (clients, channels, pollfd entries).
- Close the server socket.
- Print a "Closing server" message in red.

### SIGINT Handling Example

**main.cpp:**

```cpp
#include "./inc/Server.hpp"
#include <signal.h>
#include <cstdlib>
#include <sstream>

// Global pointer to the server instance.
Server *g_server = NULL;

// SIGINT handler for graceful shutdown.
void sigint_handler(int signum) {
    (void)signum;
    if (g_server) {
        g_server->closeServer();
    }
    exit(0);
}

int main(int argc, char **argv)
{
    int port;
    if (argc != 3) {
        std::cerr << "Bad amount of arguments! Need 2 -> <port> <password>" << std::endl;
        return EXIT_FAILURE;
    }

    std::string pswrd = argv[2];
    std::istringstream port_input(argv[1]);
    if (!(port_input >> port) || !port_input.eof() || port < 1024 || port > 65535) {
        std::cout << "Invalid input for port number!" << std::endl;
        return EXIT_FAILURE;
    }

    try {
        Server server(argv[1], argv[2]);
        g_server = &server;
        signal(SIGINT, sigint_handler);
        server.startServer();
        return 0;
    } catch (const std::exception &ex) {
        std::cerr << ex.what() << std::endl;
        return 1;
    }
}
```

**closeServer() Method (in Server.cpp):**

```cpp
void Server::closeServer() {
    // Print "Closing server" in red.
    std::cout << "\033[0;31mClosing server\033[0m" << std::endl;
    irc_log("Closing server");
    
    // Close all client connections and free memory.
    for (std::map<int, Client *>::iterator it = _clients.begin(); it != _clients.end(); ++it) {
        int client_fd = it->first;
        Client *client = it->second;
        if (client->getChannel()) {
            client->getChannel()->removeClient(client);
        }
        close(client_fd);
        delete client;
    }
    _clients.clear();
    
    // Close all poll file descriptors.
    for (std::vector<pollfd>::iterator it = _pollfds.begin(); it != _pollfds.end(); ++it) {
        close(it->fd);
    }
    _pollfds.clear();
    
    // Close the server socket.
    close(_socket);
    
    // Free all channels.
    for (std::vector<Channel *>::iterator it = _channels.begin(); it != _channels.end(); ++it) {
        delete *it;
    }
    _channels.clear();
    _running = 0;
}
```

---

## Protocol References

- **RFC 2812 – Internet Relay Chat: Client Protocol:**  
  [RFC 2812](https://www.rfc-editor.org/rfc/rfc2812.html)
- **Modern IRC Documentation – Connection Registration:**  
  [Modern IRC Docs](https://modern.ircdocs.horse/#connection-registration)

---

## Contributing

Contributions are welcome! Please follow these steps:
1. Fork the repository.
2. Create a new branch for your feature or bugfix.
3. Submit a pull request with detailed descriptions of your changes.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---
