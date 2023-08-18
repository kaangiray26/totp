# totp
Share files via P2P using TOTP verification

### Working principle
1. Open up the website on your device
2. Select file or write down your message you want to share
3. The sender creates a crypt. random RFC 6238 TOTP code for the broker connection
4. The sender uses the timed based password as its id when creating the peer
5. Recreate or change the peer id if the TOTP code changes
6. Now, the recipient must use the 6 digit TOTP code as the remote peer id to establish the broker connection
7. Once the broker connection is established, quickly switch to a secure connection based on another crypt. random uuid
8. When the actual connection is established transfer the content
