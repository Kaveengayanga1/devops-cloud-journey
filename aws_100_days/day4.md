## Allocate an Elastic IP and name it `datacenter-eip`

To allocate the Elastic IP and name it `datacenter-eip` in a single step, run the following command:

```bash
aws ec2 allocate-address \
	--domain vpc \
	--tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=datacenter-eip}]'
```

Verification

To verify the IP was allocated and named correctly, run:

```bash
aws ec2 describe-addresses --filters "Name=tag:Name,Values=datacenter-eip"
```

This will output the details of your new Elastic IP, including its `AllocationId` and `PublicIp`.

Notes on quoting (Windows `cmd.exe` vs Bash/PowerShell):
- In Bash (Linux/macOS) the single-quoted `--tag-specifications '...` works as shown.
- In Windows PowerShell use double quotes for the argument, for example:

```powershell
aws ec2 allocate-address --domain vpc --tag-specifications "ResourceType=elastic-ip,Tags=[{Key=Name,Value=datacenter-eip}]"
```

Replace the tag name or values as needed for your environment.

