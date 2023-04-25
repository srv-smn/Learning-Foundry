# Foundry

### Resources

Youtube: [smart contract programmer](https://youtube.com/playlist?list=PLO5VPQH6OWdUrKEWPF07CSuVm3T99DQki)

Github: https://github.com/t4sk/hello-foundry

Foundry Doc: [https://book.getfoundry.sh/](https://book.getfoundry.sh/)

**Installation**: 

`curl -L [https://foundry.paradigm.xyz](https://foundry.paradigm.xyz/) | bash
foundryup`

**Initialising Foundry:** `forge init`

**Test Contract:** `forge test`

**Test contract with details: `forge test -vvvv`** 

**v**, **-verbosity**...

Verbosity of the EVM.

Pass multiple times to increase the verbosity (e.g. -v, -vv, -vvv).

Verbosity levels:

- 2: Print logs for all tests

- 3: Print execution traces for failing tests

- 4: Print execution traces for all tests, and setup traces for failing tests

- 5: Print execution and setup traces for all tests

**compile contracts:** `forge build`

**Running specific Test file:** `forge test —match-path filepath_from_root.t.sol`

**Gas Report:**  `forge test —match-path filepath_from_root.t.sol --gas-report`    

**Format code:** `forge fmt`

### Test

1. `setUp()` function will be called before every test function.
2. all test function must be `public` or `external` and they should be prefixed with `test` .
3. Example Test file

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.18;

import "forge-std/Test.sol";
import {Counter} from "../src/Counter.sol";

contract CounterTest is Test {
    Counter public counter;

    // Invoked before each test
    function setUp() public {
        counter = new Counter();
    }

    // Test must be external or public
		// Name must start with test
    function testInc() public {
        counter.inc();
        assertEq(counter.count(), 1);
    }
		
		// If the name start with `test` followed by `Fail` then 
		// solidity test is expecting this function to fail
    function testFailDec() public {
        // This will fail with underflow
        // count = 0 --> count -= 1
        counter.dec();
    }

    // Same as testFailDec
		// We can simulate the above case explicitilly with
		// more specific error
    function testDecUnderflow() public {
        vm.expectRevert(stdError.arithmeticError);
        counter.dec();
    }

    function testDec() public {
        counter.inc();
        counter.inc();
        counter.dec();
        assertEq(counter.count(), 1);
    }
}
```

### solidity - version and optimisation

In the `foundry.toml` file add 

```solidity
solc_version = "0.8.18"
optimizer = true
optimizer_runs = 200
```

Link to all [config](https://github.com/foundry-rs/foundry/tree/master/config)

### Importing/installing modules

**There are 2 ways to install module**

1. **Install through forge**
    
    **To install module:** `forge install module_name`
    
    example: `forge install rari-capital/solmate`
    
    **To remove module:** `forge remove module_name`
    
    example: `forge remove solmate`
    
    **To update module:** `forge update lib/module_name`
    
    example: `forge update lib/solmate`
    
    The module will be installed inside `lib` folder.
    
    **Important**: When we install any module through `forge` it creates an automatic `remapping`.
    
    What is `remapping` ?
    
    remapping simply means when we use some word in foundry that will point to some other word/path/action. It is like a mapping of key and value.
    
    **How to import file when we install module through forge?**
    
    `import "solmate/tokens/ERC20.sol";` 
    
    We are able to directly import it as forge implicitly created a mapping for us.
    
    To view the remapping we can use the following command: `forge remappings` 
    
    O/P will be something like this:
    
     <img width="855" alt="image" src="https://user-images.githubusercontent.com/47235134/234219565-d1ff2b7e-179d-4e8e-9e91-b741c4a0c880.png">

    
2. **Install through npm**
    
    To install module we can use npm commands: `npm i package_name` 
    
    example: `npm i @openzeppelin/contracts`
    
    How to import file:
    
    `import "@openzeppelin/contracts/access/Ownable.sol";`
    
    The above line will not work, since we did not install through `forge` so remapping are not created, as remapping does not exist so the compiler is not able to figure out where he need to go when  `@openzeppelin` is mentioned in import statement.
    
    **How to create a remapping file manually?**
    
    create a `remappings.txt` file in the root folder of the project and add the remapping in form of key value form.
    
    `@openzeppelin/=node_modules/@openzeppelin`
    
    Now we can import it normally.
    
    Example:
    
    ```solidity
    pragma solidity 0.8.18;
    
    // Test import solmate
    import "solmate/tokens/ERC20.sol";
    
    contract Token is ERC20("name", "symbol", 16) {}
    
    // Test import openzeppelin
    import "@openzeppelin/contracts/access/Ownable.sol";
    
    contract TestOZ is Ownable {}
    ```
    

### console.log

To import in any file we need to import `console.sol` file.

`import "forge-std/console.sol";`

then we can use `console.log()` in our solidity program.

There are some other functions like `console.logInt()` to log `int` variable.

If we want to see those log value then we need to run the test with `-vv` 

`forge test --match-path test/Console.t.sol -vv`

 

### Authentication

If we want to test the contract with multiple addresses we need to change the `msg.sender`

To change this we can use `vm.prank(address)` , in the next link after this line msg.sender for the external contract call will become `address` that we passed in `vm.prank` .

If we want to continue this behaviour for multiple line of codes then we can use

`vm.startPrank(address)` and `vm.stopPrank()` . All the lines between these statement will have `address` as `msg.sender`  for all external contract call.

example:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.18;

import "forge-std/Test.sol";
import {Wallet} from "../src/Wallet.sol";

// forge test --match-path test/Auth.t.sol -vvvv

contract AuthTest is Test {
    Wallet public wallet;

    function setUp() public {
        wallet = new Wallet();
    }

    function testSetOwner() public {
        wallet.setOwner(address(1));
        assertEq(wallet.owner(), address(1));
    }

    function testFailNotOwner() public {
        // next call will be called by address(1)
        vm.prank(address(1));
        wallet.setOwner(address(1));
    }

    function testFailSetOwnerAgain() public {
        // msg.sender = address(this)
        wallet.setOwner(address(1));

        // Set all subsequent msg.sender to address(1)
        vm.startPrank(address(1));

        // all calls made from address(1)
        wallet.setOwner(address(1));
        wallet.setOwner(address(1));
        wallet.setOwner(address(1));

        // Reset all subsequent msg.sender to address(this)
        vm.stopPrank();

        console.log("owner", wallet.owner());

        // call made from address(this) - this will fail
        wallet.setOwner(address(1));

        console.log("owner", wallet.owner());
    }
}
```

### Test Errors

- `vm.expectRevert`
- `require` error message
- custom error
- label assertions

Example:

`contract`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract Error {
    error NotAuthorized();

    function throwError() external {
        require(false, "not authorized");
    }

    function throwCustomError() external {
        revert NotAuthorized();
    }
}
```

`testfile`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.18;

import "forge-std/Test.sol";
import {Error} from "../src/Error.sol";

contract ErrorTest is Test {
    Error public err;

    function setUp() public {
				// will be called before every test function
        err = new Error();
    }

    function testFail() public {
				// the test suit already knows that this will fail since
				// function name is `testFail`
        err.throwError();
    }

    function testRevert() public {
				// we can get the exact same behaviour as above function 
				// without keeping the name as `testFail`
				// the next function call below vm.expectRevert is expected to 
				// be reverted
        vm.expectRevert();
        err.throwError();
    }

    function testRequireMessage() public {
				// check if the code reverted with the specific error msg or not
        vm.expectRevert(bytes("not authorized"));
        err.throwError();
    }

    function testCustomError() public {
				//check if the code reverted with the specific custom error or not
        vm.expectRevert(Error.NotAuthorized.selector);
        err.throwCustomError();
    }

    // Add label to assertions
    function testErrorLabel() public {
			// when any of these assertion will fail, we will which one of 
			// these assert failed, as these label will be displayed
        assertEq(uint256(1), uint256(1), "test 1");
        assertEq(uint256(1), uint256(1), "test 2");
        assertEq(uint256(1), uint256(1), "test 3");
        assertEq(uint256(1), uint256(1), "test 4");
        assertEq(uint256(1), uint256(1), "test 5");
    }
}
```

### Test Events

`event` can be tested using `expectEmit` function, it has few parameter like 

`expectEmit(
        //     bool checkTopic1,
	
        //     bool checkTopic2,
	
        //     bool checkTopic3,
	
        //     bool checkData
	
        // )`

where `bool checkTopic1` means if we want to match the 1st indexed value of event or not. and same with `checkTopic2` and `checkTopic3` , `checkData` means do we want to compare the rest of the value of the event .

`contract`

```solidity
pragma solidity 0.8.18;

contract Event {
    event Transfer(address indexed from, address indexed to, uint256 amount);

    function transfer(address from, address to, uint256 amount) external {
        emit Transfer(from, to, amount);
    }

    function transferMany(address from, address[] calldata to, uint256[] calldata amounts) external {
        for (uint256 i = 0; i < to.length; i++) {
            emit Transfer(from, to[i], amounts[i]);
        }
    }
}
```

`test file`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.18;

import "forge-std/Test.sol";
import {Event} from "../src/Event.sol";

// forge test --match-path test/Event.t.sol -vvvv

contract EventTest is Test {
    Event public e;

    event Transfer(address indexed from, address indexed to, uint256 amount);

    function setUp() public {
        e = new Event();
    }

    function testEmitTransferEvent() public {
        // function expectEmit(
        //     bool checkTopic1,
        //     bool checkTopic2,
        //     bool checkTopic3,
        //     bool checkData
        // ) external;

        // 1. Tell Foundry which data to check
        // Check index 1, index 2 and data
        vm.expectEmit(true, true, false, true);
        // 2. Emit the expected event
        emit Transfer(address(this), address(123), 456);
        // 3. Call the function that should emit the event
        e.transfer(address(this), address(123), 456);

        // Check only index 1
        vm.expectEmit(true, false, false, false);
        emit Transfer(address(this), address(123), 456);
        // NOTE: index 2 and data (amount) doesn't match
        //       but the test will still pass
        e.transfer(address(this), address(111), 222);
    }

    function testEmitManyTransferEvent() public {
        address[] memory to = new address[](2);
        to[0] = address(111);
        to[1] = address(222);

        uint256[] memory amounts = new uint[](2);
        amounts[0] = 1;
        amounts[1] = 2;

        for (uint256 i = 0; i < to.length; i++) {
            // 1. Tell Foundry which data to check
            vm.expectEmit(true, true, false, true);
            // 2. Emit the expected event
            emit Transfer(address(this), to[i], amounts[i]);
        }

        // 3. Call the function that should emit the event
        e.transferMany(address(this), to, amounts);
    }
}
```
