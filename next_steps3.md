This is a fantastic foundation for a Mahjong scoring engine. You’ve successfully tackled the hardest part: the recursive decomposition of standard hands.

To take **Agari** from a "basic calculator" to a "professional-grade engine," there are several critical gaps—primarily around the **Kan (Quad)** mechanic, **Irregular Hands**, and **Game State** (Honba/Sticks).

Here is your actionable, step-by-step roadmap to completing the engine.

---

### Phase 1: Handling Kans (Quads)

Currently, your engine only knows about sequences and triplets. Kans are the "high-stakes" version of triplets and are the primary way players reach high Fu counts.

* **Step 1.1: Update the `Meld` Enum**
In `src/hand.rs`, add `Kantsu` to the `Meld` enum. You need to track if it is **Open** or **Closed**, as this drastically changes the Fu value (a closed terminal Kan is 32 Fu, the highest single-meld value).
* **Step 1.2: Update `calculate_fu**`
In `src/scoring.rs`, update `meld_fu` to handle `Kantsu`.
* *Simples:* Open (8), Closed (16).
* *Terminals/Honors:* Open (16), Closed (32).


* **Step 1.3: Update `decompose_hand**`
Since a Kan uses 4 tiles but counts as a 3-tile meld for hand structure, your decomposition logic needs to account for the "extra" tile used when a hand contains 15 or more tiles (due to Kans).

---

### Phase 2: Irregular Winning Shapes

Your current engine assumes every win follows the "4 Melds + 1 Pair" or "Chiitoitsu" rule. You are missing the most famous hand in the game.

* **Step 2.1: Implement Kokushi Musou (Thirteen Orphans)**
In `src/hand.rs`, add a `HandStructure::KokushiMusou` variant. Update `decompose_hand` to check for one of each terminal and honor tile plus one duplicate.
* **Step 2.2: Implement the "Max Score" Interpretation Rule**
Mahjong rules state that if a hand can be interpreted in two ways, the **highest-scoring** version must be used. You already sort your results, but you need to ensure that different wait types (e.g., a hand that could be a `Tanki` wait or a `Ryanmen` wait) are both considered so the engine can pick the one that yields more Han or Fu.

---

### Phase 3: Completing the Yaku List

Your current `yaku.rs` is missing several common and high-value patterns.

* **Step 3.1: Add Triplets & Kans Yaku**
* `Sanshoku Doukou` (Three Colored Triplets).
* `Sankantsu` (Three Quads).
* `San Ankou` (Three Concealed Triplets) - *Note: Your current logic is a bit simplified; it needs to strictly check if the triplets were completed by draw or discard.*


* **Step 3.2: Add High-Tier Yakuman**
* `Daisangen` (Big Three Dragons).
* `Shousuushii` / `Daisuushii` (Four Winds).
* `Tsuuiso` (All Honors).
* `Chinroutou` (All Terminals).



---

### Phase 4: Game Environment & "Honba"

In a real match, the score is rarely just the basic point value.

* **Step 4.1: Track Honba (Repeat Counters)**
In `src/context.rs`, add a `honba` field (u8). Every Honba adds **300 points** to the total (divided among players for Tsumo, or paid in full by the discarder for Ron).
* **Step 4.2: Track Riichi Sticks**
Add `riichi_sticks` to `GameContext`. If a player wins, they collect all 1,000-point sticks currently on the table.
* **Step 4.3: Double Wind Fu Rule**
In `src/scoring.rs`, ensure that a pair of the player's Seat Wind that is **also** the Round Wind (e.g., East player in East round) provides **4 Fu** (though some house rules say 2, 4 is the standard professional rule).

---

### Phase 5: Robust Validation

* **Step 5.1: Cross-Meld Validation**
Update `validate_hand` in `src/parse.rs`. A hand is currently "valid" if no tile appears more than 4 times, but it doesn't check if the *entire game* is valid (e.g., if you have a meld of 111m and a pair of 11m, you've used five 1-mans).
* **Step 5.2: Furiten Detection**
This is advanced, but a scoring engine often needs to know if the win was even legal. Add a `furiten` flag to the context to prevent `Ron` wins when the player has already discarded their winning tile.

---

### Recommended Next Step

Would you like me to provide the **Rust implementation for the `Kantsu` (Quad) logic** first, as it has the biggest impact on the scoring math?