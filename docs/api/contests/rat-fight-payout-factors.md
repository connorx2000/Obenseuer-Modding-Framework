---
title: Rat fight payout factors
parent: Contests
grand_parent: Player life and skills
nav_order: 3
---

# Rat fight payout factors

A payout factor is a number that your mod multiplies a rat fight's winnings by. `1` pays what the game normally pays, `2` pays double, `0.5` pays half. The factor belongs to one arena, so you need the rat fight's `ContestInfo.Id`. You get that id from `Started` or `Active()` on [`Contests`](reading-contests.md).

## Things to know

- The factor changes what the player collects, not the bet. A lost bet stays lost. The player collects the scaled winnings the usual way at the arena. <span class="tech">(The framework scales `RatFightArena.winAmount` at the end of `EndRound`. The game's own `CollectWinnings` then pays that amount.)</span>
- If the scaled winnings are zero, the arena goes straight back to betting. It behaves as it does after a loss, so the player can bet again. `Finished` still says `PlayerWon` is `true`, because the factor changes the amount, not who won.
- The game's gambling statistics keep the unscaled win. The game records the round before the factor is applied, so the totals the game keeps do not change. Only the amount the player collects does.
- If two mods set a factor for the same arena, the newest call wins. The framework writes one warning to the [log](../../guides/getting-started/logging.md) for each pair of mods and arena. If a mod calls `SetPayoutFactor` again for the same arena, it replaces its own factor with no warning.
- `Clear(modId)` takes back your mod's factors on every arena. It returns `true` when it removed at least one. It returns `false` for a bad [mod id](../../glossary.md#mod-id) or when your mod had none.

## Limits

- `SetPayoutFactor` returns `false` when it did not set the factor. That happens for a bad mod id and for a factor below 0 or above 4. It also happens for an id that does not match an arena in the [scene](../../glossary.md#scene) you are in. And it happens when payout factors are not set up. A factor from 0 to 4 is accepted. It also works while a save is loading, because the factors are not saved. <span class="tech">(A mod id must be 1 to 64 characters of letters, digits, `.`, `_` or `-`.)</span>
- The factors are forgotten when the game loads a new main scene. That happens whenever a [save](../../guides/getting-started/save-data.md) is loaded or the player goes back to the main menu, because arena ids belong to one scene. Set them again after each load. <span class="tech">(They are cleared on every `LoadSceneMode.Single` scene load.)</span>

**Common problems**

- `SetPayoutFactor` returns `false` and nothing changes: the mod id is bad, the factor is outside 0 to 4, or the id is not an arena in the current scene. Check `PayoutFactorsAvailable` too. When it is `false`, the payout change failed to set up, see [Details](details.md#if-something-fails-to-set-up).
- A factor stops working after the player loads a save or returns to the main menu: the factors are cleared with the scene. Set them again after each load.
