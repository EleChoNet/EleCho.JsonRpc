# EleCho.JsonRpc [![](https://img.shields.io/badge/-����-green)](README.md) [![](https://img.shields.io/badge/-English-green)](README.en.md)

���� JSON �ļ� RPC ��. \
Simple JSON based RPC library.

> ͨ���Ķ�����Ŀ�Ĵ���, �����ѧ��: ��̬����. ��Ŀ��Ҫ�߼����벻���� 300 ��. \
> By reading the code of this project, you can learn: Dynamic proxy. The main logic code of the project does not exceed 300 lines.

## ���� / Transmission

```txt
--> ��ͷ(���ֽ�����) + {"Method":"������","Arg":["����"]}
<-- ��ͷ(���ֽ�����) + {"Ret":"����ֵ","RefRet":["���÷���ֵ"],"Err":"������Ϣ"}
```
```txt
--> header(four-byte integer) + {"Method":"method name","Arg":["arguments"]}
<-- header(four-byte integer) + {"Ret":"return value","RefRet":["reference returns"],"Err":"error message"}
```

> ע: ��������ȷ��Ӧ����ֵʱ, Err �ֶ�Ӧ��Ϊ null \
> Note: The Err field should be null when the method responds correctly with the return value

## ʹ�� / Usage

�ÿ������ `System.IO.Stream` ��ʹ�� \
This library can be used on `System.IO.Stream`

���幫���Ľӿ�(Define the public interface):

```csharp
public interface Commands
{
    public void WriteLine(string message);
    public int Add(int a, int b);
    public int Add114514(ref int num);
}
```

����˶Խӿڵ�ʵ��(Server implementation of the interface):

```csharp
internal class CommandsImpl : Commands
{
    public int Add(int a, int b) => a + b;
    public int Add114514(ref int num) => num += 114514;
    public void WriteLine(string message) => Console.WriteLine("Server print: " + message);
}
```

����˼��� TCP (Server listening on TCP):

```csharp
int port = 11451;

TcpListener listener = new TcpListener(new IPEndPoint(IPAddress.Any, port));      // ����ָ���˿� / listen on specified port
listener.Start();

CommandsImpl serverCommands = new CommandsImpl();                                 // �������õ�ָ�����ʵ�� / Create a common command call instance
List<RpcServer<Commands>> rpcs = new List<RpcServer<Commands>>();                 // �������пͻ��� RPC ���� / Save all client RPC references

Console.WriteLine($"Listening {port}");

while (true)
{
    TcpClient client = await listener.AcceptTcpClientAsync();                     // ����һ���ͻ��� / Accept a client
    rpcs.Add(new RpcServer<Commands>(client.GetStream(), serverCommands));        // ���������� RPC ʵ�� / Create and save an RPC instance
}
```

�ͻ������Ӳ�����Զ�̺���(The client connects and calls the remote function):

```csharp
Console.Write("Addr: ");
var addr = Console.ReadLine()!;                         // �û������ַ / User enters the address

TcpClient client = new TcpClient();
client.Connect(IPEndPoint.Parse(addr));                 // ���ӵ������� / Connect to server

RpcClient<Commands> rpc =
    new RpcClient<Commands>(client.GetStream());        // ���� RPC �ͻ���ʵ�� / Create an RPC client instance

int num = 10;
rpc.Remote.Add114514(ref num);

if (num == 114524)
    Console.WriteLine("�� ref ������ RPC ���óɹ�");

while (true)
{
    var input = Console.ReadLine();
    if (input == null)
        break;

    rpc.Remote.WriteLine(input);                        // ���÷���� WriteLine ���� / Call the server WriteLine method
}
```

> �ͻ��˿���̨(Client console): \
> Addr: 127.0.0.1:11451 \
> �� ref ������ RPC ���óɹ�\
> hello \
> this message is from client

> ����˿���̨: \
> Listening 11451 \
> Server print: hello \
> Server print: this message is from client