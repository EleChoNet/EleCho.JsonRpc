# EleCho.JsonRpc

���� JSON �ļ� RPC ��.

> ͨ���Ķ�����Ŀ�Ĵ���, �����ѧ��: ��̬����. ��Ŀ��Ҫ�߼����벻���� 250 ��.

## �����ʽ

```txt
--> ��ͷ(���ֽ�����) + {"Method":"������","Arg":["����"]}
<-- ��ͷ(���ֽ�����) + {"Ret":"����ֵ","Err":"������Ϣ"}
```

> ע: ��������ȷ��Ӧ����ֵʱ, Err �ֶ�Ӧ��Ϊ null

## ʹ��

�ÿ������ `System.IO.Stream` ��ʹ��

���幫���Ľӿ�:

```csharp
public interface Commands
{
    public void WriteLine(string message);
    public int Add(int a, int b);
}
```

����˶Խӿڵ�ʵ��:

```csharp
internal class CommandsImpl : Commands
{
    public int Add(int a, int b) => a + b;
    public void WriteLine(string message) => Console.WriteLine("Server print: " + message);
}
```

����˼��� TCP:

```csharp
int port = 11451;

TcpListener listener = new TcpListener(new IPEndPoint(IPAddress.Any, port));      // ����ָ���˿�
listener.Start();

CommandsImpl serverCommands = new CommandsImpl();                                 // �������õ�ָ�����ʵ��
List<RpcServer<Commands>> rpcs = new List<RpcServer<Commands>>();                 // �������пͻ��� RPC ����

Console.WriteLine($"Listening {port}");

while (true)
{
    TcpClient client = await listener.AcceptTcpClientAsync();                     // ����һ���ͻ���
    rpcs.Add(new RpcServer<Commands>(client.GetStream(), serverCommands));        // ���������� RPC ʵ��
}
```

�ͻ������Ӳ�����Զ�̺���:

```csharp
Console.Write("Addr: ");
var addr = Console.ReadLine()!;                         // �û������ַ

TcpClient client = new TcpClient();
client.Connect(IPEndPoint.Parse(addr));                 // ���ӵ�������

RpcClient<Commands> rpc =
    new RpcClient<Commands>(client.GetStream());        // ���� RPC �ͻ���ʵ��

while (true)
{
    var input = Console.ReadLine();
    if (input == null)
        break;

    rpc.Remote.WriteLine(input);                        // ���÷���� WriteLine ����
}
```

> �ͻ��˿���̨: \
> Addr: 127.0.0.1:11451 \
> hello \
> this message is from client

> ����˿���̨: \
> Listening 11451 \
> Server print: hello \
> Server print: this message is from client