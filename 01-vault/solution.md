This vulnerability is well know in the crypto security ecosystem and is refered as the Vault Inflation Attack. It was first identified on November 15th, 2022, during a security assessment conducted by OpenZeppelin. Since then, several mitigation strategies have been introduced to protect against this flaw in the ERC-4626 tokenized vault standard.

The vault inflation attack targets ERC-4626 contracts, which are designed to represent yield-bearing vaults. In these contracts, shares are minted when users deposit assets, granting them rights to the vault's underlyig assets. As these assets are reinvested in other DeFi protocols, they generate yield, which subsequently increases the value of each share. Users can later redeem their shares when withdrawing from the vault, collecting the accrued yield.

The attack specifically targets newly deployed vault contracts, allowing malicious actors to fully or partially steal the initial deposits into the contracts due to a rounding issue in the share calculation.

To execute the attack, an attacker needs to perform two transactions. First, when the vault is deployed, the attacker deposits a small amount into it to ensure he gets at least 1 share from the vault. Once the attacker detects a large initial deposit, he frontruns this transaction by depositing the same amount just before the victim's deposit.

The deposit function uses the formula mintedShares = totalSupply() * assets / totalAssets(). Due to Solidity's integer division rounding down, the victim receives 0 shares. Meanwhile, the attacker can then redeem his share and steal the victim's entire deposit.

A clear example of this attack is documented in the MixBytes blog post on ERC 4626 inflation attack:

    Attack scenario:

        A hacker back-runs the transaction of an ERC4626 pool creation.
        The hacker mints for themself one share: deposit(1). Thus, totalAsset()==1, totalSupply()==1.
        The hacker front-runs the deposit of the victim who wants to deposit 20,000 USDT (20,000.000000).
        The hacker inflates the denominator right in front of the victim: asset.transfer(20_000e6). Now totalAsset()==20_000e6 + 1, totalSupply()==1.
        Next, the victim's tx takes place. The victim gets 1 * 20_000e6 / (20_000e6 + 1) == 0 shares. The victim gets zero shares.
        The hacker burns their share and gets all the money.

Attack Mechanics

    - Initial Setup by Attacker: The attacker first deposits a minimal amount of assets (e.g., 1 token) into an empty vault, receiving a small number of shares in return. The attacker then artificially inflates the vault's asset pool by directly donating a large number of tokens (e.g., 100,000 tokens). This shifts the exchange rate significantly, making it very high.
    - Exchange Rate Manipulation: After the donation, the vault now has a large amount of assets relative to the small number of shares issued. This creates a scenario where any subsequent deposit would result in fewer shares being issued, due to the high exchange rate.
    - User Deposit and Loss: If a user attempts to deposit assets into the vault after this manipulation, the high exchange rate means their deposit would result in a calculation of shares that rounds down to zero, or a negligible amount. Essentially, the user receives little to no shares in return for their deposit.
    - Attacker's Gain: As the only significant shareholder, the attacker can then withdraw almost all the assets from the vault, effectively stealing the user's deposit.


OpenZeppelin introduced ERC4626 in v4.7.0 (Jun 30 2022) and added mitigation measure to inflation attack in v4.9.0 (May 23 2023) (https://github.com/OpenZeppelin/openzeppelin-contracts/releases?page=2).