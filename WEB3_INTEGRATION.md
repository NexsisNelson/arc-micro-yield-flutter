# Web3 Integration Guide for Arc Micro-Yield Flutter App

## Overview

This guide covers integrating Web3 functionality for blockchain interactions with the Arc Micro-Yield smart contracts.

## Setup

### Dependencies

The project already includes:
```yaml
web3dart: ^3.0.2
```

You may need additional packages:
```bash
flutter pub add http
flutter pub add provider  # For state management
flutter pub add shared_preferences  # For local storage
```

## Basic Web3 Integration

### 1. Initialize Web3 Client

```dart
import 'package:web3dart/web3dart.dart';
import 'package:http/http.dart';

class Web3Service {
  late Web3Client _ethClient;
  late DeployedContract _vault;
  late DeployedContract _strategyManager;

  Future<void> initialize() async {
    // Connect to Arc blockchain RPC
    _ethClient = Web3Client(
      'https://arc-testnet-rpc.example.com',
      Client(),
    );

    // Load contract ABI
    _vault = await _loadContract('MicroYieldVault');
    _strategyManager = await _loadContract('StrategyManager');
  }

  Future<DeployedContract> _loadContract(String name) async {
    // Load ABI from assets
    final abi = // Load from JSON file
    return DeployedContract(
      ContractAbi.fromJson(abi, name),
      EthereumAddress.fromHex('0x...contract-address...'),
    );
  }
}
```

### 2. Read Contract Data

```dart
// Get vault balance
Future<BigInt> getBalance(String userAddress) async {
  final function = _vault.function('balanceOf');
  final result = await _ethClient.call(
    contract: _vault,
    function: function,
    params: [EthereumAddress.fromHex(userAddress)],
  );
  return result[0] as BigInt;
}

// Get total assets
Future<BigInt> getTotalAssets() async {
  final function = _vault.function('totalAssets');
  final result = await _ethClient.call(
    contract: _vault,
    function: function,
  );
  return result[0] as BigInt;
}

// Get accumulated yield
Future<BigInt> getAccumulatedYield(String strategyAddress) async {
  final function = _strategyManager.function('accumulatedYield');
  final result = await _ethClient.call(
    contract: _strategyManager,
    function: function,
    params: [EthereumAddress.fromHex(strategyAddress)],
  );
  return result[0] as BigInt;
}
```

### 3. Write Transactions

```dart
// Deposit USDC
Future<String> deposit(
  String userPrivateKey,
  BigInt amount,
) async {
  final function = _vault.function('deposit');
  final credential = EthPrivateKey.fromHex(userPrivateKey);

  final tx = await _ethClient.sendTransaction(
    credential,
    Transaction.callContract(
      contract: _vault,
      function: function,
      parameters: [amount, EthereumAddress.fromHex(userAddress)],
      gasPrice: EtherAmount.inWei(BigInt.from(1000000000)), // 1 Gwei
      maxGas: 200000,
    ),
  );

  return tx;
}

// Withdraw from vault
Future<String> withdraw(
  String userPrivateKey,
  BigInt shares,
) async {
  final function = _vault.function('redeem');
  final credential = EthPrivateKey.fromHex(userPrivateKey);

  final tx = await _ethClient.sendTransaction(
    credential,
    Transaction.callContract(
      contract: _vault,
      function: function,
      parameters: [shares, EthereumAddress.fromHex(userAddress), EthereumAddress.fromHex(userAddress)],
      maxGas: 200000,
    ),
  );

  return tx;
}

// Allocate to strategy
Future<String> allocateToStrategy(
  String userPrivateKey,
  String strategyAddress,
  BigInt amount,
) async {
  final function = _strategyManager.function('allocateToStrategy');
  final credential = EthPrivateKey.fromHex(userPrivateKey);

  final tx = await _ethClient.sendTransaction(
    credential,
    Transaction.callContract(
      contract: _strategyManager,
      function: function,
      parameters: [
        EthereumAddress.fromHex(strategyAddress),
        amount,
      ],
      maxGas: 300000,
    ),
  );

  return tx;
}
```

### 4. Listen for Events

```dart
// Listen for Deposit events
Stream<FilterEvent> watchDepositEvents() {
  final filter = FilterOptions.events(
    contract: _vault,
    event: _vault.event('Deposit'),
  );

  return _ethClient.events(filter);
}

// Listen for Withdraw events
Stream<FilterEvent> watchWithdrawEvents() {
  final filter = FilterOptions.events(
    contract: _vault,
    event: _vault.event('Withdrawal'),
  );

  return _ethClient.events(filter);
}

// Parse events
void _subscribeToEvents() {
  watchDepositEvents().listen((event) {
    // Parse event data
    final indexed = event.topics;
    final data = event.data;
    print('Deposit: $data');
  });
}
```

## State Management Integration

### Using Provider

```dart
class Web3Provider extends ChangeNotifier {
  late Web3Service _web3;
  BigInt _userBalance = BigInt.zero;
  BigInt _totalAssets = BigInt.zero;
  bool _isLoading = false;

  BigInt get userBalance => _userBalance;
  BigInt get totalAssets => _totalAssets;
  bool get isLoading => _isLoading;

  Future<void> loadData(String userAddress) async {
    _isLoading = true;
    notifyListeners();

    try {
      _userBalance = await _web3.getBalance(userAddress);
      _totalAssets = await _web3.getTotalAssets();
    } catch (e) {
      print('Error loading data: $e');
    }

    _isLoading = false;
    notifyListeners();
  }

  Future<void> deposit(String privateKey, BigInt amount) async {
    _isLoading = true;
    notifyListeners();

    try {
      await _web3.deposit(privateKey, amount);
      // Wait for confirmation, then reload data
      await Future.delayed(Duration(seconds: 5));
      await loadData(userAddress);
    } catch (e) {
      print('Deposit failed: $e');
    }

    _isLoading = false;
    notifyListeners();
  }
}
```

### Using Provider in UI

```dart
class DashboardScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<Web3Provider>(
      builder: (context, web3Provider, child) {
        if (web3Provider.isLoading) {
          return Center(child: CircularProgressIndicator());
        }

        return Column(
          children: [
            Text('Balance: \${web3Provider.userBalance}'),
            Text('Total Assets: \${web3Provider.totalAssets}'),
            ElevatedButton(
              onPressed: () async {
                await web3Provider.deposit(privateKey, amount);
              },
              child: Text('Deposit'),
            ),
          ],
        );
      },
    );
  }
}
```

## Security Considerations

### Private Key Management

⚠️ **NEVER** store private keys in plain text!

```dart
// Use secure storage
final secureStorage = FlutterSecureStorage();

// Store encrypted
await secureStorage.write(
  key: 'private_key',
  value: encryptedPrivateKey,
);

// Retrieve encrypted
final encryptedKey = await secureStorage.read(key: 'private_key');
final privateKey = decrypt(encryptedKey);
```

### Transaction Signing

```dart
// Always validate before signing
Future<String> secureTransaction(
  String privateKey,
  Function buildTx,
) async {
  // Validate private key format
  if (!_isValidPrivateKey(privateKey)) {
    throw Exception('Invalid private key');
  }

  // Build and review transaction
  final tx = buildTx();

  // Sign with private key
  final credential = EthPrivateKey.fromHex(privateKey);
  return await _ethClient.sendTransaction(credential, tx);
}
```

## Testing

### Mock Web3 for Testing

```dart
class MockWeb3Service implements Web3Service {
  @override
  Future<BigInt> getBalance(String address) async {
    return BigInt.from(1000000); // 1 USDC (with 6 decimals)
  }

  @override
  Future<BigInt> getTotalAssets() async {
    return BigInt.from(1000000000);
  }
}
```

### Test Example

```dart
test('Deposit increases balance', () async {
  final web3 = MockWeb3Service();
  final initialBalance = await web3.getBalance(address);

  await web3.deposit(privateKey, BigInt.from(100000));
  final newBalance = await web3.getBalance(address);

  expect(newBalance, greaterThan(initialBalance));
});
```

## Contract ABI Files

Store contract ABIs in `assets/contracts/`:

```json
// assets/contracts/MicroYieldVault.json
{
  "contractName": "MicroYieldVault",
  "abi": [
    {
      "name": "deposit",
      "type": "function",
      "inputs": [
        {"name": "assets", "type": "uint256"},
        {"name": "receiver", "type": "address"}
      ],
      "outputs": [{"name": "shares", "type": "uint256"}]
    }
  ]
}
```

## Environment Configuration

Create `.env` file for network configuration:

```env
# Arc Testnet
ARC_TESTNET_RPC=https://arc-testnet-rpc.example.com
VAULT_ADDRESS=0x...
STRATEGY_MANAGER_ADDRESS=0x...

# Arc Mainnet
ARC_MAINNET_RPC=https://arc-mainnet-rpc.example.com

# Local Testing
LOCAL_RPC=http://localhost:8545
```

Load in app:

```dart
final rpcUrl = dotenv.env['ARC_TESTNET_RPC']!;
_ethClient = Web3Client(rpcUrl, Client());
```

## Error Handling

```dart
try {
  await web3.deposit(privateKey, amount);
} on JsonRPC2Exception catch (e) {
  print('RPC Error: ${e.message}');
} on RPCError catch (e) {
  print('Transaction failed: ${e.message}');
} catch (e) {
  print('Unexpected error: $e');
}
```

## Resources

- [web3dart Documentation](https://pub.dev/packages/web3dart)
- [Ethereum JSON-RPC API](https://ethereum.org/en/developers/docs/apis/json-rpc/)
- [ERC-4626 Vault Standard](https://eips.ethereum.org/EIPS/eip-4626)
- [Arc Blockchain Documentation](https://docs.arcblockchain.io)

---

Next steps: Implement UI screens for deposit, withdraw, and portfolio tracking!
