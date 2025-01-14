# loam-sdk

- [Subcontracts](#subcontracts)
    - [Creating Contract Subcontracts](#creating-contract-subcontracts)
    - [External API](#external-api)
- [Core](#core-subcontract)
-   [Using the Core Subcontract](#using-the-core-subcontract)


# Subcontracts

A subcontract is a type that implements the `IntoKey` trait, which is used for lazily loading and storing the type.

## Creating  Subcontracts

Here's an example of how to create a subcontract:

```rust
#[contracttype]
#[derive(IntoKey)]
pub struct Messages(Map<Address, String>);
```

This generates the following implementation:

```rust
impl IntoKey for Messages {
    type Key = IntoVal<Env, RawVal>;
    fn into_key() -> Self::Key {
      String::from_slice("messages")
    }
```

## External API

You can also create and implement external APIs for contract subcontracts:

```rust
#[subcontract]
pub trait IsPostable {
    fn messages_get(&self, author: Address) -> Option<String>;
    fn messages_set(&mut self, author: Address, text: String);
}
```

# Core Subcontract

The `Core` trait provides the minimum logic needed for a contract to be redeployable. A contract should be able to be redeployed to another contract that can also be redeployed. Redeployment requires admin status, as it would be undesirable for an account to redeploy the contract without permission.

## Using  `Core`

To use the core subcontract, create a `Contract` struct and implement `Core` for it. This makes `Contract` redeployable by the Admin of the contract and will continue to be redeployable if the new contract also implements `Core`. After `Core` other Subcontracts can be added as needed.

```rust
use loam_sdk::{soroban_contract, soroban_sdk};
use loam_subcontract_core::{Admin, Core};

pub struct Contract;

impl Core for Contract {
    type Impl = Admin;
}

soroban_contract!();
```

This code generates the following implementation:

```rust
struct SorobanContract;

#[contractimpl]
impl SorobanContract {
     pub fn admin_set(env: Env, admin: Address) {
        set_env(env);
        Contract::owner_set(owner);
    }
    pub fn admin_get(env: Env) -> Option<Address> {
        set_env(env);
        Contract::admin_get()
    }
    pub fn redeploy(env: Env, wasm_hash: BytesN<32>) {
        set_env(env);
        Contract::redeploy(wasm_hash);
    }
    // Subcontract methods would be inserted here.
    // Contract must implement all Subcontracts and is the proxy for the contract calls.
    // This is because the Subcontracts have default implementations which call the associated type
}
```

By specifying the associated `Impl` type for `Core`, you enable the default `Admin` methods to be used (`admin_set`, `admin_get`, `redeploy`). However, you can also provide a different implementation if needed by replacing `Admin` with a different struct/enum that also implements [IsCore](replace).

Notice that the generated code calls `Contract::redeploy` and other methods. This ensures that the `Contract` type is redeployable, while also allowing for extensions, as `Contract` can overwrite the default methods.
