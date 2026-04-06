#![no_std]
use soroban_sdk::{contract, contractimpl, contracttype, vec, Address, Env, Vec, Map};

#[derive(Clone)]
#[contracttype]
pub enum DataKey {
    Participants,
    Balances, // Map<Address, i128>
}

#[contract]
pub struct ExpenseSplitter;

#[contractimpl]
impl ExpenseSplitter {
    // Initialize the group with a list of participants
    pub fn init(env: Env, participants: Vec<Address>) {
        env.storage().instance().set(&DataKey::Participants, &participants);
        
        let mut balances: Map<Address, i128> = Map::new(&env);
        for p in participants.iter() {
            balances.set(p, 0);
        }
        env.storage().instance().set(&DataKey::Balances, &balances);
    }

    // Add an expense: 'payer' paid 'amount', to be split among all participants
    pub fn add_expense(env: Env, payer: Address, amount: i128) {
        payer.require_auth();

        let participants: Vec<Address> = env.storage().instance().get(&DataKey::Participants).unwrap();
        let mut balances: Map<Address, i128> = env.storage().instance().get(&DataKey::Balances).unwrap();
        
        let share = amount / (participants.len() as i128);

        for p in participants.iter() {
            let current_balance = balances.get(p.clone()).unwrap();
            if p == payer {
                // Payer gets credit for what others owe (Total - their own share)
                balances.set(p, current_balance + (amount - share));
            } else {
                // Others owe their share
                balances.set(p, current_balance - share);
            }
        }

        env.storage().instance().set(&DataKey::Balances, &balances);
    }

    // Fetch current balances
    pub fn get_balances(env: Env) -> Map<Address, i128> {
        env.storage().instance().get(&DataKey::Balances).unwrap()
    }
}   


1. Contract address: CAVBK7WQXIRYL24K5FUO2AYMCFUHLGU3SHVPAEHYI4M4C4HF4RTT2RME
2. Wallet address:   GAFB465D2PDIPEN3A7IHUKC3BFHBBNASTP44LCRFE6EUZ7VG7PWELSJ2
3. Image:<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2b13c818-eb55-4de5-9eaf-af0a2fe55f79" />

4. Link: https://stellar.expert/explorer/testnet/contract/CAVBK7WQXIRYL24K5FUO2AYMCFUHLGU3SHVPAEHYI4M4C4HF4RTT2RME
