# sss-rs
[![Build Status](https://travis-ci.com/bilowik/sss-rs.svg?branch=master)](https://travis-ci.com/bilowik/sss-rs)

An implementation of a secret sharing scheme in Rust. 

# Some things to note:
	- Depending on the configuration you decide, the shares may not be the same size as the secret,
	  but this is okay. 

	- Currently has a relatively high memory footprint when working with large secrets. This is planned
	  to be fixed in 0.3.2/0.4.0

		- To continue giving the raw_share API as much flexibility as possible, this will likely be 
		  implemented for the Sharer API only to give devs the choice in how they want to handle memory
		  usage when using the raw_share API.

	- Works on single bytes of data, meaning the Prime must be greater than 255. In the future this may 
	  change to allow working on N bytes of data at a time to improve performance.

Not intended to be used in production code.

# New Example with the current API
```
let dir = "./";
let stem = "test";
let num_shares = 3;
let secret: Vec<u8> = vec![5, 4, 9, 1, 2, 128, 43];
let sharer = Sharer::builder(secret)
	.shares_required(num_shares)
	.shares_to_create(num_shares)
	.coefficient_bits(32)
	.build()
	.unwrap();
sharer.share_to_files(dir, stem).unwrap();
let recon = Sharer::reconstructor(dir, stem, num_shares, PrimeLocation::Default).unwrap();
assert_eq!(secret, *recon_secret);


```


# Old Example (makes use of raw_share functions)
```
let mut rand = SmallRng::seed_from_u64(123u64);
let secret: u8 = 23; // The secret to be split into shares
let shares_required = 3; // The number of shares required to reconstruct the secret
let shares_to_create = 3; // The number of shares to create, can be greater than the required
let bit_size_co: usize = rand.gen_range(32, 65); // The number of bits for the generated coefficients
let prime_bits: usize = rand.gen_range(bit_size_co + 128, 257); // The number of bits for the prime
let mut prime: BigInt = rand.gen_prime(prime_bits).into(); // The prime number used for finite field
while prime < BigUint::from(secret) {
	// In case the prime is less than the secret, generate new ones until one is greater
	prime = rand.gen_prime(prime_bits).into();
}

let shares: Vec<Point> = create_shares_from_secret(	secret,
							&prime
							shares_required,
							shares_to_create,
							bit_size_co).unwrap();
let secret_recon = reconstruct_secret(shares, &prime, shares_required).unwrap();

assert_eq!(secret, secret_recon);
```

# TODO:
	- Currently limited to 4GB files, and memory usage issues especially when working with many shares.
		- Break up the reading in from files in chunks of N bytes to avoid high memory usage and overcome
		  the file size limitation.
	- The shuffle operation was left out of the new API, mainly because it would not function after the 
	  above change is implemented. May re-implement a way to shuffle the data in-file. For now this 
	  will be benched.
	- Add optional verification that allows for checking if a secret has been properly reconstructed.
		- One way to do this would be to take the hash of the first N bytes of the secret and place it at		   the end of the secret. The chances of incorrect reconstruction leading to the first N bytes 
		  hashing to the incorrectly reconstructed hash placed at the end would be astronomically low.
	- While working with individual bytes makes things much easier, it would be far more efficient to 
	  work with larger chunks of data.
