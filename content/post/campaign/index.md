---
title: 提案投票合约

description: 一个竞选学生会主席场景的智能合约

slug: campaign

date: 2022-09-28

image: header.jpg

categories: 
 - Contract

tags:
 - Solidity
---
### 合约介绍

#### 场景介绍

campaign 合约，实现场景为学生会主席竞选流程。

#### 流程介绍

1. 合约初始化时，指定具有owner权限的账户，该账户具有检票与提名候选人以及启动竞选提案的权利。
2. 启动竞选提案，需要提名一位候选人，并指定提案通过票数阈值，更新提案状态为投票中，返回该次提案ID。
3. 任何人都可以根据提案ID查询该提案的状态。
4. 当提案状态为投票中，任何人可以调用投票功能进行投票，限投一人一票，重复投票不会生效。
5. 当票数大于启动提案时设置的阈值，提案状态为待检票，owner 可启动检票。
6. 检票会进行身份验证，只有owner账户可以成功调用。
7. 检票通过后，提案流程结束，提案状态置为已生效，此时任何人都可以查询当前学生会主席信息。
8. 当提案状态为未启动、已生效、待检票时，不可使用投票功能。
9. 任何人都可根据提案ID查询该提案的候选人信息。

#### 功能介绍

##### constructor

- Params:
  - owner:
    - type: address
    - desc: owner权限设置
- Desc: 设置owner权限账户，初始化提案状态

##### propose

- Params:
  - candidate:
    - type: string
    - desc: 候选人
  - target:
    - type: uint256
    - desc: 票数阈值
- Returns: 
  - proposalId:
    - type: uint256
    - desc: 提案ID
- Desc: 启动竞选提案

##### queryProposal

- Params:
  - proposalId:
    - type: uint256
    - desc: 提案ID

- Returns:
  - status:
    - type: string
    - desc: 提案状态：未启动(stoped)、投票中(voting)、待检票(passed)、已生效(completed)
- Desc: 查询提案状态

##### vote

- Params:
  - proposalId:
    - type: uint256
    - desc: 提案ID

- Retruns:
  - status:
    - type: string
    - desc: 投票状态："success" or "error"，error需给出原因
- Desc: 进行投票，调用该方法，视为调用者进行投票一次

##### queryVotes

- Params:
  - proposalId:
    - type: uint256
    - desc: 提案ID

- Returns:
  - votes:
    - type: uint256
    - desc: 当前所获票数，当状态为未启动时，应为0
- Desc: 查询已经获取的票数

##### check

- Params:
  - proposalId:
    - type: uint256
    - desc: 提案ID

- Returns:
  - status:
    - type: string
    - desc: 检票状态："success" or "error"，error需给出原因
- Desc: 启动检票

##### queryPresident

- Returns:
  - president:
    - type: string
    - desc: 竞选成功的候选人
- Desc: 查询学生会主席信息

##### queryCandidate

- Params:
  - proposalId: 
    - type: uint256
    - desc: 提案ID
- Returns:
  - candidate:
    - type: string
    - desc: 提案候选人信息
- Desc: 查询提案候选人信息

### 合约设计

> SafeMath 可提供。

```solidity
// SPDX-License-Identifier: SimPL-2.0

pragma solidity ^0.8.0;

contract campaign{

	// 为 uint256 类型使用 SafeMath 库
	using SafeMath for uint256;

	// owner 账户
	address _owner;

	// 学生会主席
	string _president;

	// 提案候选人
	mapping(uint256 => string) _candidate;

	// 提案票数
	mapping(uint256 => uint256) _votes;

	// 票数阈值
	uint256 _votesTarget;
	
	// 提案ID
	uint256 _proposalId;
	
	// 提案状态 为了减少 gas 费消耗，使用uint256类型进行状态记录，并通过mapping来提供状态映射
	mapping(uint256 => uint256) proposalStatus;
	mapping(uint256 => string) pstatus;

	// 投票者集合
	mapping(uint256 => mapping(address => bool)) voted;


	/*
	* 初始化函数
	*/
	constructor(address owner){
		_owner = owner;
		// 初始化提案状态映射
		pstatus[0] = "stoped";
		pstatus[1] = "voting";
		pstatus[2] = "passed";
		pstatus[3] = "completed";
	}


	/*
	* 启动竞选提案 theSender() 方法非必须，等同于 msg.sender;
	*/
	function propose(string memory candidate, uint256 target) public returns(uint256 proposalId){
		// 验证身份
		require(theSender() == _owner, "error: No permission");
		// 验证提案状态
		require(proposalStatus[_proposalId] == 0, "error: The proposal is not stoped status");
		// 设置候选人
		_candidate[_proposalId] = candidate;
		// 设置票数阈值
		_votesTarget = target;
		// 设置提案状态
		proposalStatus[_proposalId] = 1;
		
		return _proposalId;
	}


	/*
	* 查询提案状态
	*/
	function queryProposal(uint256 proposalId) public view returns(string memory status){
		return pstatus[proposalStatus[proposalId]];
	}


	/*
	* 投票
	*/
	function vote(uint256 proposalId) public returns(string memory status){
		// 验证提案状态
		require(proposalStatus[proposalId] == 1, "error: The proposal is not voteable");
		// 验证是否已投票
		require(voted[proposalId][theSender()] == false, "error: Each person can only vote once");
		// 增加已获取的票数
		_votes[proposalId] = _votes[proposalId].add(1);
		// 避免重复投票
		voted[proposalId][theSender()] = true;
		// 是否达到票数阈值
		if (_votes[proposalId] >= _votesTarget){
			proposalStatus[proposalId] = 2;
		}
		return "vote successfuly";
	}


	/*
	* 查询提案获取的票数
	*/
	function queryVotes(uint256 proposalId) public view returns(uint256 votes){
		return _votes[proposalId];
	}


	/*
	* 检票
	*/
	function check(uint256 proposalId) public returns(string memory status){
		// 验证提案状态
		require(proposalStatus[proposalId] == 2, "error: The proposal is not eligible for wicket");
		// 验证调用者身份
		require(theSender() == _owner, "error: No permission");
		// 修改提案状态
		proposalStatus[proposalId] = 3;
		// 修改学生会主席信息
		_president = _candidate[proposalId];
		// 为下一次发起提案做准备
		_proposalId = _proposalId.add(1);
		return "check successfuly, the proposal will be take effect";
	}


	/*
	* 查询学生会主席信息
	*/
	function queryPresident() public view returns(string memory president){
		return _president;
	}

	
	/*
	* 查询提案的候选人信息
	*/
	function queryCandidate(uint256 proposalId) public view returns(string memory candidate){
		return _candidate[proposalId];
	}
	

	/*
	* 返回此次调用者
	*/
	function theSender() private view returns(address){
		return msg.sender;
	}
}

library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    // Gas optimization: this is cheaper than requiring 'a' not being zero, but the
    // benefit is lost if 'b' is also tested.
    // See: https://github.com/OpenZeppelin/openzeppelin-solidity/pull/522
    if (a == 0) {
      return 0;
    }

    uint256 c = a * b;
    require(c / a == b);

    return c;
  }

  /**
  * @dev Integer division of two numbers truncating the quotient, reverts on division by zero.
  */
  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b > 0); // Solidity only automatically asserts when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold

    return c;
  }

  /**
  * @dev Subtracts two numbers, reverts on overflow (i.e. if subtrahend is greater than minuend).
  */
  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b <= a);
    uint256 c = a - b;

    return c;
  }

  /**
  * @dev Adds two numbers, reverts on overflow.
  */
  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    require(c >= a);

    return c;
  }

  /**
  * @dev Divides two numbers and returns the remainder (unsigned integer modulo),
  * reverts when dividing by zero.
  */
  function mod(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b != 0);
    return a % b;
  }

}
```

### 合约部署

#### 编译

**`solc --abi --bin ./campaign.sol -o build`**

#### 部署

> 通过 Java SDK 在百度超级链开放网络进行部署。
>
> SDK使用文档：[Java SDK接入指南](https://xuper.baidu.com/n/xuperdoc/xuperos/xuperos_java.html)

##### 获取SDK

```xml
<dependency>
  <groupId>com.baidu.xuper</groupId>
  <artifactId>xuper-java-sdk</artifactId>
  <version>0.3.0</version>
</dependency>
```

##### 部署合约

```java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.AddressTrans;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
      	AddressTrans owner = AddressTrans.xChainToEvmAddress(account.getAddress());
        client = new XuperClient("39.156.69.83:37100");
        // 开放网络工作台注册的合约账户
        String contractAccount = "";
        account.setContractAccount(contractAccount);
        try {
            // 合约编译文件
            byte[] abi = Files.readAllBytes(Paths.get("./build/campaign.abi"));
            byte[] bin = Files.readAllBytes(Paths.get("./build/campaign.bin"));

            Map<String,String> params = new HashMap<>();
            params.put("owner", owner.getAddr());

            Transaction tx = client.deployEVMContract(account,bin,abi,"campaign", params);
            System.out.println(tx.getContractResponse().getBodyStr());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

### 合约调用

#### 发起提案

```java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

import java.math.BigInteger;
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
        client = new XuperClient("39.156.69.83:37100");
        String contract = "campaign";
        String method = "propose";
        Map<String,String> params = new HashMap<>();
        params.put("candidate", "alice");
        params.put("target", "2");
        Transaction tx = client.invokeEVMContract(account,contract, method, params, BigInteger.ZERO);
        System.out.println(tx.getContractResponse().getBodyStr());
    }
}

// output
[{"proposalId":"0"}]
```

#### 查询提案状态

```java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

import java.util.HashMap;
import java.util.Map;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
        client = new XuperClient("39.156.69.83:37100");
        String contract = "campaign";
        String method = "queryProposal";
        Map<String,String> params = new HashMap<>();
        params.put("proposalId", "0");
        Transaction tx = client.queryEVMContract(account,contract, method, params);
        System.out.println(tx.getContractResponse().getBodyStr());
    }
}

// output
[{"status":"voting"}]
```

#### 投票

``` java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

import java.math.BigInteger;
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
        Account account2 = Account.getAccountFromFile("开放网络私钥目录","安全码");
        client = new XuperClient("39.156.69.83:37100");
        String contract = "campaign";
        String method = "vote";
        Map<String,String> params = new HashMap<>();
        params.put("proposalId", "0");
        // 投两票
        Transaction tx = client.invokeEVMContract(account,contract, method, params, BigInteger.ZERO);
        System.out.println("account vote res: " + tx.getContractResponse().getBodyStr());
      	// 重复投票
      	Transaction tx = client.invokeEVMContract(account,contract, method, params, BigInteger.ZERO);
        System.out.println("account vote res: " + tx.getContractResponse().getBodyStr());
        Transaction tx2 = client.invokeEVMContract(account2,contract, method, params, BigInteger.ZERO);
        System.out.println("account2 vote res: " + tx2.getContractResponse().getBodyStr());
      	// 非投票状态下投票
      	Transaction tx2 = client.invokeEVMContract(account2,contract, method, params, BigInteger.ZERO);
        System.out.println("account2 vote res: " + tx2.getContractResponse().getBodyStr());
    }
}

// output
account vote res: [{"status":"vote successfuly"}]
'error: Each person can only vote once'
account2 vote res: [{"status":"vote successfuly"}]
'error: The proposal is not voteable'
```

#### 查询已经获取的票数

```java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

import java.util.HashMap;
import java.util.Map;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
        client = new XuperClient("39.156.69.83:37100");
        String contract = "campaign";
        String method = "queryVotes";
        Map<String,String> params = new HashMap<>();
        params.put("proposalId", "0");
        Transaction tx = client.queryEVMContract(account,contract, method, params);
        System.out.println(tx.getContractResponse().getBodyStr());
    }
}

// output
[{"votes":"2"}]
```

#### 检票

```java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

import java.math.BigInteger;
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
        client = new XuperClient("39.156.69.83:37100");
        String contract = "campaign";
        String method = "check";
        Map<String,String> params = new HashMap<>();
        params.put("proposalId", "0");
        Transaction tx = client.invokeEVMContract(account,contract, method, params, BigInteger.ZERO);
        System.out.println(tx.getContractResponse().getBodyStr());
    }
}

// output
[{"status":"check successfuly, the proposal will be take effect"}]
```

#### 查询学生会主席信息

```java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
        client = new XuperClient("39.156.69.83:37100");
        String contract = "campaign";
        String method = "queryPresident";
        Transaction tx = client.queryEVMContract(account,contract, method, null);
        System.out.println(tx.getContractResponse().getBodyStr());
    }
}

// output
[{"president":"alice"}]
```

#### 查询候选人信息

```java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

import java.util.HashMap;
import java.util.Map;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
        client = new XuperClient("39.156.69.83:37100");
        String contract = "campaign";
        String method = "queryCandidate";
        Map<String,String> params = new HashMap<>();
        params.put("proposalId", "0");
        Transaction tx = client.queryEVMContract(account,contract, method, params);
        System.out.println(tx.getContractResponse().getBodyStr());
    }
}

// output
[{"candidate":"alice"}]
```

#### 创建新一轮提案

```java
import com.baidu.xuper.api.Account;
import com.baidu.xuper.api.Transaction;
import com.baidu.xuper.api.XuperClient;
import com.baidu.xuper.config.Config;

import java.math.BigInteger;
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static XuperClient client;
    public static void main(String[] args) {
        // 设置背书服务
        Config.setConfigPath("./conf/sdk.yaml");
        Account account = Account.getAccountFromFile("开放网络私钥目录","安全码");
        client = new XuperClient("39.156.69.83:37100");
        String contract = "campaign";
        String method = "propose";
        Map<String,String> params = new HashMap<>();
        params.put("candidate", "bob");
        params.put("target", "3");
        Transaction tx = client.invokeEVMContract(account,contract, method, params, BigInteger.ZERO);
        System.out.println(tx.getContractResponse().getBodyStr());
    }
}

// output
[{"proposalId":"1"}]
```



