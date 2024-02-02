# foundry tutorial
 Berikut adalah tutorial untuk menggunakan Foundry dalam membuat Proof of Concept (PoC) terhadap bug leads. Tutorial ini mencakup langkah-langkah instalasi, konfigurasi, penggunaan remappings, dan pengujian PoC. Tutorial ini juga memberikan petunjuk untuk membuat PoC untuk bug yang sebenarnya.
 
 ### Buat folder baru
buat folder baru silakan buat degnan nama foundrytest
 



 ###  Instalasi dan Konfigurasi Awal

Inisialisasi Proyek Foundry:


        forge init --template immunefi-team/forge-poc-templates --branch default

### atau (rekomendasi untuk uji coba pertama gunakan ini)
    
        forge init

### Konfigurasi Proyek:
Edit file foundry.toml dan tambahkan baris berikut:





        [default]
        src = 'src'
        out = 'out'
        libs = ['lib']
        chain_id = 1
        eth_rpc_url = 'https://eth-mainnet.alchemyapi.io/v2/{your_alchemy_api_key}'
        block_number = 14812830
        etherscan_api_key = '{your_etherscan_api_key}'
        

Pastikan untuk menyertakan etherscan_api_key untuk kejelasan trace saat menjalankan forge test dalam mode verbose.

# Mengambil Sumber Proyek

### Mengambil Sumber Proyek:
Jalankan perintah berikut untuk mendapatkan sumber proyek:
(lewati jika sudah installasi)

        npm i -g ethereum-sources-downloader
        
jika sudah melakukan instalasi langsung menuju step berikut 
Dapatkan sumber proyek yang dikerjakan dengan menjalankan perintah berikut, Ini akan membuat direktori lib/StakingV3 yang berisi sumber-sumber kontrak yang dikerjakan dan dependensinya.
        
        ethereum-sources-downloader etherscan 0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670 lib

### Menggunakan Remappings:
Jalankan perintah remappings:

        forge remappings
        
maka akan muncul

        StakingV3/=lib/StakingV3/
        ds-test/=lib/forge-std/lib/ds-test/src/
        forge-std/=lib/forge-std/src/

Pastikan bahwa remappings mencakup direktori StakingV3/ dan @openzeppelin/.


 ### sesuaikan versi solidity
 pada folder src,test terdapat file berbeda
 Counter.t.sol
 Counter.s.sol
 
##  Dalam struktur proyek Foundry, folder src dan test memiliki peran dan jenis file yang berberda antara Counter.t.sol dan Counter.s.sol:

### Counter.t.sol:

#### Lokasi: Biasanya berada di folder test. 
 
#### Fungsi Utama: File ini digunakan untuk menulis dan menjalankan pengujian atau tes terhadap kontrak smart dalam proyek.

### Counter.s.sol:

#### Lokasi: Biasanya berada di folder src.
 
#### Fungsi Utama: File ini digunakan untuk menuliskan dan mendefinisikan kontrak smart atau skrip yang akan dijalankan pada jaringan blockchain.

### Menggunakan Remappings
Untuk menangani masalah dengan dependensi dari @openzeppelin, gunakan fitur remappings di Foundry. Buat file remappings.txt di root proyek dan tambahkan baris berikut
selanjutnya buat fil remappings.txt dan didalamnya isi  pada baris pertama.

        //remappings.txt
        
        @openzeppelin/=lib/StakingV3/StakingV3/@openzeppelin/
        
selanjutnya adalah jalankan 

       forge test

### Hasil:

       @openzeppelin/=lib/StakingV3/StakingV3/@openzeppelin/
       StakingV3/=lib/StakingV3/
       ds-test/=lib/forge-std/lib/ds-test/src/
       forge-std/=lib/forge-std/src/

        
## sesuaikan path jika mengalami error

# Pengujian Proof of Concept
Gunakan fitur Foundry untuk mengubah state yang terkait dengan tampilan kontrak. Berikut contoh token dalam setup tes:
bah pada Counter.t.sol

    // SPDX-License-Identifier: UNLICENSED
    pragma solidity ^0.8.0;

    import "forge-std/Test.sol";
    import "StakingV3/StakingV3/contracts/staking/StakingV3/StakingV3.sol";
    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
    contract CounterTest is Test {
    IERC20 yopToken = IERC20(0xAE1eaAE3F627AAca434127644371b67B18444051);
    StakingV3 staking = StakingV3(0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670);
    address attacker = address(1);
    using stdStorage for StdStorage;

    function writeTokenBalance(
        address who,
        IERC20 token,
        uint256 amt
    ) internal {
        stdstore
            .target(address(token))
            .sig(token.balanceOf.selector)
            .with_key(who)
            .checked_write(amt);
    }
    function setUp() public {
	    writeTokenBalance(attacker, yopToken, 500 ether);
    }
        function testExample() public {
        vm.startPrank(attacker);
        uint8 lock_duration_months = 1;
        yopToken.approve(address(staking), 500 ether);
        staking.stake(500 ether, lock_duration_months);
        }
       }


### Menjalankan Test:
Jalankan test dengan perintah:

       forge test


output

      [⠑] Compiling...
      [⠘] Compiling 18 files with 0.8.24
      [⠒] Compiling 59 files with 0.8.9
      [⠃] Solc 0.8.24 finished in 8.19s
      [⠑] Solc 0.8.9 finished in 11.98s
      Compiler run successful with warnings:
      Warning (5667): Unused function parameter. Remove or comment out the variable name to silence this warning.
         --> lib/StakingV3/StakingV3/contracts/staking/StakingV3/StakingV3Base.sol:261:5:
          |
      261 |     address _operator,
          |     ^^^^^^^^^^^^^^^^^


      Running 1 test for test/Counter.t.sol:CounterTest
      [PASS] testExample() (gas: 336381)
      Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 42.87s

Pastikan untuk menyesuaikan alamat token dan kontrak staking sesuai dengan proyek yang sedang diuji.

atau gunakan perintah dengan output lebih detail dengan tambahan -v,-vv,-vvv,-vvvv(maksimal adalah 4 v)
contoh

      forge test -vvvv

### Output:


      [⠰] Compiling...
      No files changed, compilation skipped

      Running 1 test for test/Counter.t.sol:CounterTest
      [PASS] testExample() (gas: 336381)
      Traces:
        [336381] CounterTest::testExample()
    ├─ [0] VM::startPrank(0x0000000000000000000000000000000000000001)
    │   └─ ← ()
    ├─ [24562] YOP::approve(ERC1967Proxy: [0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670], 500000000000000000000 [5e20])
    │   ├─ emit Approval(owner: 0x0000000000000000000000000000000000000001, spender: ERC1967Proxy: [0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670], value: 500000000000000000000 [5e20])
    │   └─ ← true
    ├─ [320925] ERC1967Proxy::stake(500000000000000000000 [5e20], 1)
    │   ├─ [316024] StakingV3::stake(500000000000000000000 [5e20], 1) [delegatecall]
    │   │   ├─ [2531] YOP::balanceOf(0x0000000000000000000000000000000000000001) [staticcall]
    │   │   │   └─ ← 500000000000000000000 [5e20]
    │   │   ├─ [29858] AccessControlManager::hasAccess(0x0000000000000000000000000000000000000001, ERC1967Proxy: [0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670]) [staticcall]
    │   │   │   ├─ [8684] SanctionsListAccessControl::blockedAccess(0x0000000000000000000000000000000000000001, ERC1967Proxy: [0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670]) [staticcall]
    │   │   │   │   ├─ [2923] SanctionsList::isSanctioned(0x0000000000000000000000000000000000000001) [staticcall]
    │   │   │   │   │   └─ ← false
    │   │   │   │   └─ ← false
    │   │   │   ├─ [5003] AllowAnyAccessControl::hasAccess(0x0000000000000000000000000000000000000001, ERC1967Proxy: [0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670]) [staticcall]
    │   │   │   │   └─ ← true
    │   │   │   └─ ← true
    │   │   ├─ [98455] ERC1967Proxy::calculateStakingRewards(237)
    │   │   │   ├─ [93560] YOPRewardsV2::calculateStakingRewards(237) [delegatecall]
    │   │   │   │   ├─ [899] ERC1967Proxy::totalWorkingSupply() [staticcall]
    │   │   │   │   │   ├─ [504] StakingV3::totalWorkingSupply() [delegatecall]
    │   │   │   │   │   │   └─ ← 1653128834947760 [1.653e15]
    │   │   │   │   │   └─ ← 1653128834947760 [1.653e15]
    │   │   │   │   ├─ [1448] ERC1967Proxy::workingBalanceOfStake(237) [staticcall]
    │   │   │   │   │   ├─ [1050] StakingV3::workingBalanceOfStake(237) [delegatecall]
    │   │   │   │   │   │   └─ ← 0
    │   │   │   │   │   └─ ← 0
    │   │   │   │   └─ ← ()
    │   │   │   └─ ← ()
    │   │   ├─ [13655] YOP::transferFrom(0x0000000000000000000000000000000000000001, ERC1967Proxy: [0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670], 500000000000000000000 [5e20])
    │   │   │   ├─ emit Transfer(from: 0x0000000000000000000000000000000000000001, to: ERC1967Proxy: [0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670], value: 500000000000000000000 [5e20])
    │   │   │   ├─ emit Approval(owner: 0x0000000000000000000000000000000000000001, spender: ERC1967Proxy: [0x5B705d7c6362A73fD56D5bCedF09f4E40C2d3670], value: 0)       
    │   │   │   └─ ← true
    │   │   ├─ emit TransferSingle(operator: 0x0000000000000000000000000000000000000001, from: 0x0000000000000000000000000000000000000000, to: 0x0000000000000000000000000000000000000001, id: 237, value: 1)
    │   │   ├─ emit Staked(_user: 0x0000000000000000000000000000000000000001, _tokenId: 237, _amount: 500000000000000000000 [5e20], _lockPeriod: 1, _startTime: 1706880347 [1.706e9])
    │   │   └─ ← 237
    │   └─ ← 237
    └─ ← ()

      Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 38.30s

      Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)


### forge help:
bantuan command gunakan

        forge help
      
maka akan muncul

        forge help
        Build, test, fuzz, debug and deploy Solidity contracts

        Usage: forge <COMMAND>

        Commands:
          bind               Generate Rust bindings for smart contracts
          build              Build the project's smart contracts [aliases: b, compile]
          cache              Manage the Foundry cache
          clean              Remove the build artifacts and cache directories [aliases: cl]
          completions        Generate shell completions script [aliases: com]
          config             Display the current config [aliases: co]
          coverage           Generate coverage reports
          create             Deploy a smart contract [aliases: c]
          debug              Debugs a single smart contract as a script [aliases: d]
          doc                Generate documentation for the project
          flatten            Flatten a source file and all of its imports into one file [aliases: f]
          fmt                Format Solidity source files
          geiger             Detects usage of unsafe cheat codes in a project and its dependencies
          generate           Generate scaffold files
          generate-fig-spec  Generate Fig autocompletion spec [aliases: fig]
          help               Print this message or the help of the given subcommand(s)
          init               Create a new Forge project
          inspect            Get specialized information about a smart contract [aliases: in]
          install            Install one or multiple dependencies [aliases: i]
          remappings         Get the automatically inferred remappings for the project [aliases: re]
          remove             Remove one or multiple dependencies [aliases: rm]
          script             Run a smart contract as a script, building transactions that can be  sent onchain
          selectors          Function selector utilities [aliases: se]
          snapshot           Create a snapshot of each test's gas usage [aliases: s]
          test               Run the project's tests [aliases: t]
          tree               Display a tree visualization of the project's dependency graph [aliases: tr]
          update             Update one or multiple dependencies [aliases: u]
          verify-check       Check verification status on Etherscan [aliases: vc]
          verify-contract    Verify smart contracts on Etherscan [aliases: v]

        Options:
          -h, --help     Print help
          -V, --version  Print version

        Find more information in the book: http://book.getfoundry.sh/reference/forge/forge.html

