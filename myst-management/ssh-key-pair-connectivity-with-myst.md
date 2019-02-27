Setting up SSH connectivity using username and password or SSH keypairs? Let's start by making you aware of how MyST connects via SSH.

## SSH Connectivity between Servers

|    # | Source      | Destination        |
| ---: | ----------- | ------------------ |
|    1 | MyST        | AdminServer        |
|    2 | AdminServer | AdminServer        |
|    3 | AdminServer | Managed Server 1   |
|    4 | AdminServer | Managed Server 2   |
|    5 | AdminServer | Managed Server ... |

# Testing Connectivity

When running SSH commands to test connectivity ensure **each command does not prompt and is passwordless**.

*NOTE: You may need to accept the **fingerprint** the first time. Once accepted, repeat the tests and ensure this time they are passwordless.*

### SSH Username and Password

1. Test the connection #1 from MyST
   1. `ssh oracle@AdminServer`
2. Test connection #2 from AdminServer to AdminServer
   1. `ssh oracle@AdminServer`
3. Test connection #3 from AdminServer to Managed Server 1
   1. `ssh oracle@ManagedServer1`
4. Test connection # from AdminServer to Managed Server 2
   1. `ssh oracle@ManagedServer2`
5. And so on...

### SSH Keypair

In this example you will need access to the `id_rsa` public key.

1. Test the connection #1 from MyST
   1. `ssh -i id_rsa oracle@AdminServer`
2. Test connection #2 from AdminServer to AdminServer
   1. `ssh -i id_rsa oracle@AdminServer`
3. Test connection #3 from AdminServer to Managed Server 1
   1. `ssh -i id_rsa oracle@ManagedServer1`
4. Test connection # from AdminServer to Managed Server 2
   1. `ssh -i id_rsa oracle@ManagedServer2`
5. And so on...