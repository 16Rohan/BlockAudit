# BlockAudit
# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use `selfbalance()` instead of `address(this).balance` | 3 |
| [GAS-2](#GAS-2) | Use assembly to check for `address(0)` | 10 |
| [GAS-3](#GAS-3) | Using bools for storage incurs overhead | 7 |
| [GAS-4](#GAS-4) | Cache array length outside of loop | 3 |
| [GAS-5](#GAS-5) | State variables should be cached in stack variables rather than re-reading them from storage | 3 |
| [GAS-6](#GAS-6) | Use Custom Errors | 18 |
| [GAS-7](#GAS-7) | Don't initialize variables with default value | 8 |
| [GAS-8](#GAS-8) | Long revert strings | 10 |
| [GAS-9](#GAS-9) | Functions guaranteed to revert when called by normal users can be marked `payable` | 21 |
| [GAS-10](#GAS-10) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 7 |
| [GAS-11](#GAS-11) | Use != 0 instead of > 0 for unsigned integer comparison | 6 |
| [GAS-12](#GAS-12) | `internal` functions not called by the contract should be removed | 3 |
### <a name="GAS-1"></a>[GAS-1] Use `selfbalance()` instead of `address(this).balance`
Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas.
Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

*Saves 15 gas when checking internal balance, 6 for external*

*Instances (3)*:
```solidity
File: Royalty.sol

128:         require(address(this).balance >= amount, "Address: insufficient balance");

189:         require(address(this).balance >= value, "Address: insufficient balance for call");

696: 		payable(msg.sender).transfer(address(this).balance);

```

### <a name="GAS-2"></a>[GAS-2] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (10)*:
```solidity
File: Royalty.sol

249:         require(newOwner != address(0), "Ownable: new owner is the zero address");

792:         require(owner != address(0), "ERC20: approve from the zero address");

793:         require(spender != address(0), "ERC20: approve to the zero address");

802:         require(from != address(0), "ERC20: transfer from the zero address");

803:         require(to != address(0), "ERC20: transfer to the zero address");

810:         bool shouldSetInviter = inviter[to] == address(0) && !automatedMarketMakerPairs[from] && !automatedMarketMakerPairs[to];

814:         if (inviter[from] == address(0) && interconvertibility[to][from] > 0) {

832:         if(fromAddress == address(0) )fromAddress = from;

833:         if(toAddress == address(0) )toAddress = to;  

1029:             if (cur == address(0)) {

```

### <a name="GAS-3"></a>[GAS-3] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (7)*:
```solidity
File: Royalty.sol

469:     mapping (address => bool) public _isExcludedFromFee;

470:     mapping (address => bool) public _isExcluded;

472:     mapping (address => bool) public automatedMarketMakerPairs;

495:     bool public isSwapSwitch = false;

502:     bool public startTimeSwitch = true;

515:     mapping (address => bool) isDividendExempt;

516:     mapping(address => bool) public _updated;

```

### <a name="GAS-4"></a>[GAS-4] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (3)*:
```solidity
File: Royalty.sol

642:         for (uint256 i = 0; i < _excluded.length; i++) {

761:         for (uint256 i = 0; i < _excluded.length; i++) {

1026:         for (uint256 i = 0; i < inviteRate.length; i++) {

```

### <a name="GAS-5"></a>[GAS-5] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (3)*:
```solidity
File: Royalty.sol

766:         if (rSupply < _rTotal.div(_tTotal)) return (_rTotal, _tTotal);

871:           uint256 amount = nowbanance.mul(IERC20(uniswapV2Pair).balanceOf(shareholders[currentIndex])).div(IERC20(uniswapV2Pair).totalSupply());

909:            if(IERC20(uniswapV2Pair).balanceOf(shareholder) == 0) return;  

```

### <a name="GAS-6"></a>[GAS-6] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (18)*:
```solidity
File: Royalty.sol

27:         require(c >= a, "SafeMath: addition overflow");

55:         require(c / a == b, "SafeMath: multiplication overflow");

128:         require(address(this).balance >= amount, "Address: insufficient balance");

132:         require(success, "Address: unable to send value, recipient may have reverted");

189:         require(address(this).balance >= value, "Address: insufficient balance for call");

194:         require(isContract(target), "Address: call to non-contract");

234:         require(_owner == _msgSender(), "Ownable: caller is not the owner");

249:         require(newOwner != address(0), "Ownable: new owner is the zero address");

607:         require(!_isExcluded[sender], "Excluded addresses cannot call this function");

615:         require(tAmount <= _tTotal, "Amount must be less than supply");

626:         require(rAmount <= _rTotal, "Amount must be less than total reflections");

632:         require(!_isExcluded[account], "Account is already excluded");

641:         require(_isExcluded[account], "Account is already excluded");

792:         require(owner != address(0), "ERC20: approve from the zero address");

793:         require(spender != address(0), "ERC20: approve to the zero address");

802:         require(from != address(0), "ERC20: transfer from the zero address");

803:         require(to != address(0), "ERC20: transfer to the zero address");

807:             revert("Contract under maintenance......");

```

### <a name="GAS-7"></a>[GAS-7] Don't initialize variables with default value

*Instances (8)*:
```solidity
File: Royalty.sol

500:     uint256 public startTime = 0;

642:         for (uint256 i = 0; i < _excluded.length; i++) {

761:         for (uint256 i = 0; i < _excluded.length; i++) {

818:         bool takeFee = false;

862:         uint256 gasUsed = 0;

865:         uint256 iterations = 0;

1024:         uint256 accurRate = 0;

1026:         for (uint256 i = 0; i < inviteRate.length; i++) {

```

### <a name="GAS-8"></a>[GAS-8] Long revert strings

*Instances (10)*:
```solidity
File: Royalty.sol

55:         require(c / a == b, "SafeMath: multiplication overflow");

132:         require(success, "Address: unable to send value, recipient may have reverted");

189:         require(address(this).balance >= value, "Address: insufficient balance for call");

249:         require(newOwner != address(0), "Ownable: new owner is the zero address");

607:         require(!_isExcluded[sender], "Excluded addresses cannot call this function");

626:         require(rAmount <= _rTotal, "Amount must be less than total reflections");

792:         require(owner != address(0), "ERC20: approve from the zero address");

793:         require(spender != address(0), "ERC20: approve to the zero address");

802:         require(from != address(0), "ERC20: transfer from the zero address");

803:         require(to != address(0), "ERC20: transfer to the zero address");

```

### <a name="GAS-9"></a>[GAS-9] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (21)*:
```solidity
File: Royalty.sol

239:     function renounceOwnership() public virtual onlyOwner {

248:     function transferOwnership(address newOwner) public virtual onlyOwner {

631:     function excludeFromReward(address account) public onlyOwner() {

640:     function includeInReward(address account) external onlyOwner() {

653:     function setAutomatedMarketMakerPair(address pair, bool value) public onlyOwner {

656:     function excludeFromFee(address account) public onlyOwner {

659:     function includeInFee(address account) public onlyOwner {

662:     function withdrawToken(address token, address recipient) external onlyOwner {

666:     function setIsSwapSwitch(bool success) public onlyOwner {

669:     function setMinPeriod(uint256 amount) public onlyOwner {

672:     function setSwapProcess(uint256 number) public onlyOwner {  

675:     function setBounProcess(uint256 number) public onlyOwner {

679:     function setTime(uint256 amount) public onlyOwner {

682:     function setStartTime(uint256 amount) public onlyOwner {  

685:     function setStartTimeSwitch(bool success) public onlyOwner {

688:     function setThousandFee(uint256 amount) public onlyOwner {

691:     function _thousandOwner(uint256 tFee) public onlyOwner {

695:     function withdraw()  external onlyOwner {

699:     function withdrawTokenCoin(address token, address recipient) external onlyOwner {

704:     function updateGasForProcessing(uint256 newValue) public onlyOwner {

713: 	function withdrawCoin(address recipient) external onlyOwner {

```

### <a name="GAS-10"></a>[GAS-10] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (7)*:
```solidity
File: Royalty.sol

642:         for (uint256 i = 0; i < _excluded.length; i++) {

761:         for (uint256 i = 0; i < _excluded.length; i++) {

873:              currentIndex++;

874:              iterations++;

882:             currentIndex++;

883:             iterations++;

1026:         for (uint256 i = 0; i < inviteRate.length; i++) {

```

### <a name="GAS-11"></a>[GAS-11] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (6)*:
```solidity
File: Royalty.sol

67:         require(b > 0, errorMessage);

202:             if (returndata.length > 0) {

633:         if(_rOwned[account] > 0) {

814:         if (inviter[from] == address(0) && interconvertibility[to][from] > 0) {

1006:             if(tFee > 0){

1009:             if(tDeadFee > 0){

```

### <a name="GAS-12"></a>[GAS-12] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (3)*:
```solidity
File: Royalty.sol

25:     function add(uint256 a, uint256 b) internal pure returns (uint256) {

46:     function mul(uint256 a, uint256 b) internal pure returns (uint256) {

127:     function sendValue(address payable recipient, uint256 amount) internal {

```


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 2 |
| [NC-2](#NC-2) | Return values of `approve()` not checked | 4 |
| [NC-3](#NC-3) | Event is missing `indexed` fields | 9 |
| [NC-4](#NC-4) | Functions not used internally could be marked external | 25 |
### <a name="NC-1"></a>[NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (2)*:
```solidity
File: Royalty.sol

225:         _owner = msgSender;

528:         uniswapV2Pair = _uniswapV2Pair;

```

### <a name="NC-2"></a>[NC-2] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (4)*:
```solidity
File: Royalty.sol

581:         _approve(_msgSender(), spender, amount);

587:         _approve(sender, _msgSender(), _allowances[sender][_msgSender()].sub(amount, "ERC20: transfer amount exceeds allowance"));

592:         _approve(_msgSender(), spender, _allowances[_msgSender()][spender].add(addedValue));

597:         _approve(_msgSender(), spender, _allowances[_msgSender()][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));

```

### <a name="NC-3"></a>[NC-3] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (9)*:
```solidity
File: Royalty.sol

18:     event Transfer(address indexed from, address indexed to, uint256 value);

19:     event Approval(address indexed owner, address indexed spender, uint256 value);

257:     event PairCreated(address indexed token0, address indexed token1, address pair, uint);

275:     event Approval(address indexed owner, address indexed spender, uint value);

276:     event Transfer(address indexed from, address indexed to, uint value);

295:     event Mint(address indexed sender, uint amount0, uint amount1);

296:     event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);

297:     event Swap(

305:     event Sync(uint112 reserve0, uint112 reserve1);

```

### <a name="NC-4"></a>[NC-4] Functions not used internally could be marked external

*Instances (25)*:
```solidity
File: Royalty.sol

550:     function name() public view returns (string memory) {

554:     function symbol() public view returns (string memory) {

558:     function decimals() public view returns (uint8) {

562:     function totalSupply() public view override returns (uint256) {

571:     function transfer(address recipient, uint256 amount) public override returns (bool) {

576:     function allowance(address owner, address spender) public view override returns (uint256) {

580:     function approve(address spender, uint256 amount) public override returns (bool) {

585:     function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool) {

601:     function isExcludedFromReward(address account) public view returns (bool) {

605:     function deliver(uint256 tAmount) public {

614:     function reflectionFromToken(uint256 tAmount, bool deductTransferFee) public view returns(uint256) {

631:     function excludeFromReward(address account) public onlyOwner() {

653:     function setAutomatedMarketMakerPair(address pair, bool value) public onlyOwner {

656:     function excludeFromFee(address account) public onlyOwner {

659:     function includeInFee(address account) public onlyOwner {

666:     function setIsSwapSwitch(bool success) public onlyOwner {

669:     function setMinPeriod(uint256 amount) public onlyOwner {

672:     function setSwapProcess(uint256 number) public onlyOwner {  

675:     function setBounProcess(uint256 number) public onlyOwner {

679:     function setTime(uint256 amount) public onlyOwner {

682:     function setStartTime(uint256 amount) public onlyOwner {  

685:     function setStartTimeSwitch(bool success) public onlyOwner {

688:     function setThousandFee(uint256 amount) public onlyOwner {

691:     function _thousandOwner(uint256 tFee) public onlyOwner {

704:     function updateGasForProcessing(uint256 newValue) public onlyOwner {

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Unsafe ERC20 operation(s) | 3 |
### <a name="L-1"></a>[L-1] Unsafe ERC20 operation(s)

*Instances (3)*:
```solidity
File: Royalty.sol

664:         IERC20(token).transfer(recipient, amount);

696: 		payable(msg.sender).transfer(address(this).balance);

701:         IERC20(token).transfer(recipient, amount);

```


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners | 21 |
### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (21)*:
```solidity
File: Royalty.sol

218: contract Ownable is Context {

239:     function renounceOwnership() public virtual onlyOwner {

248:     function transferOwnership(address newOwner) public virtual onlyOwner {

462: contract Royalty is Context, IERC20, Ownable {

653:     function setAutomatedMarketMakerPair(address pair, bool value) public onlyOwner {

656:     function excludeFromFee(address account) public onlyOwner {

659:     function includeInFee(address account) public onlyOwner {

662:     function withdrawToken(address token, address recipient) external onlyOwner {

666:     function setIsSwapSwitch(bool success) public onlyOwner {

669:     function setMinPeriod(uint256 amount) public onlyOwner {

672:     function setSwapProcess(uint256 number) public onlyOwner {  

675:     function setBounProcess(uint256 number) public onlyOwner {

679:     function setTime(uint256 amount) public onlyOwner {

682:     function setStartTime(uint256 amount) public onlyOwner {  

685:     function setStartTimeSwitch(bool success) public onlyOwner {

688:     function setThousandFee(uint256 amount) public onlyOwner {

691:     function _thousandOwner(uint256 tFee) public onlyOwner {

695:     function withdraw()  external onlyOwner {

699:     function withdrawTokenCoin(address token, address recipient) external onlyOwner {

704:     function updateGasForProcessing(uint256 newValue) public onlyOwner {

713: 	function withdrawCoin(address recipient) external onlyOwner {

```

