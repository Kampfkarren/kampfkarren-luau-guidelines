# \[Kampfkarren's] Roblox and Luau Guidelines and Patterns

Inspired by [the C++ core guidelines](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md).

<!-- Throw the document into https://ecotrust-canada.github.io/markdown-toc/ for this -->
- [This document](#this-document)
- [General philosophy](#general-philosophy)
  * [Code should be strictly typed](#code-should-be-strictly-typed)
  * [Bugs that can be caught statically should](#bugs-that-can-be-caught-statically-should)
  * [Code should be simple](#code-should-be-simple)
  * [Prefer immutability](#prefer-immutability)
  * [Build code that works with tools](#build-code-that-works-with-tools)
  * [Developer experience takes priority over performance](#developer-experience-takes-priority-over-performance)
- [General code guidelines](#general-code-guidelines)
  * [Early return/continue when the function logically should not keep going](#early-return-continue-when-the-function-logically-should-not-keep-going)
  * [It can be okay for implementations to be messier if it improves consumer usage](#it-can-be-okay-for-implementations-to-be-messier-if-it-improves-consumer-usage)
  * [In modules, make a named variable and return it, rather than returning directly](#in-modules--make-a-named-variable-and-return-it--rather-than-returning-directly)
    + [Exception - Stories and specs](#exception---stories-and-specs)
  * [Prefer individual files for libraries and not huge utility libraries](#prefer-individual-files-for-libraries-and-not-huge-utility-libraries)
  * [Comments should describe now or the future, but not yesterday](#comments-should-describe-now-or-the-future--but-not-yesterday)
  * [Suffix yielding functions with `Async`](#suffix-yielding-functions-with--async-)
  * [Shallow, not deep copy](#shallow--not-deep-copy)
- [Luau](#luau)
  * [Avoid dynamic requires](#avoid-dynamic-requires)
  * [Prefer to avoid the idea of truthiness and falsiness in favor of explicit checks](#prefer-to-avoid-the-idea-of-truthiness-and-falsiness-in-favor-of-explicit-checks)
    + [Exception - If expressions](#exception---if-expressions)
    + [Exception - `and` and `or`](#exception----and--and--or-)
  * [Use `and` and `or` for short circuiting and defaults](#use--and--and--or--for-short-circuiting-and-defaults)
    + [Exception - Don't use `x and y or z`](#exception---don-t-use--x-and-y-or-z-)
  * [Don't hide builtins](#don-t-hide-builtins)
  * [Don't use string or table call syntax](#don-t-use-string-or-table-call-syntax)
  * [Type publicly exposed functions fully](#type-publicly-exposed-functions-fully)
  * [Aside from functions, avoid trivial types](#aside-from-functions--avoid-trivial-types)
  * [Don't use `pcall`'s shorthand syntax for methods, sometimes use it in other cases](#don-t-use--pcall--s-shorthand-syntax-for-methods--sometimes-use-it-in-other-cases)
  * [Prefer `{ [K]: V? }` over `{ [K]: V }` where invalid keys are expected to index](#prefer-----k---v-----over-----k---v----where-invalid-keys-are-expected-to-index)
  * [`nil` does not mean "nothing".](#-nil--does-not-mean--nothing-)
  * [Use string style enums, and nothing else](#use-string-style-enums--and-nothing-else)
  * [Use generalized iteration](#use-generalized-iteration)
  * [Keep optional arguments clear](#keep-optional-arguments-clear)
  * [Always give `assert` an error message.](#always-give--assert--an-error-message)
  * [`assert` should only take constant error messages](#-assert--should-only-take-constant-error-messages)
  * [Have a point to asserting `typeof`](#have-a-point-to-asserting--typeof-)
  * [Avoid metatables](#avoid-metatables)
    + [You probably don't agree with that, but let's at least agree on...](#you-probably-don-t-agree-with-that--but-let-s-at-least-agree-on)
    + [Exception - Weak tables](#exception---weak-tables)
  * [Sort requires alphabetically, and do not section them](#sort-requires-alphabetically--and-do-not-section-them)
- [Roblox](#roblox)
  * [`GetService` everything](#-getservice--everything)
  * [Use `UDim2.fromOffset` and `UDim2.fromScale`](#use--udim2fromoffset--and--udim2fromscale-)
  * [Put scripts inside scripts as implementation details](#put-scripts-inside-scripts-as-implementation-details)
  * [Use absolute paths, avoid `script.Parent` outside of implementations, and avoid going up more than one Parent](#use-absolute-paths--avoid--scriptparent--outside-of-implementations--and-avoid-going-up-more-than-one-parent)
    + [Exception - Stories and tests](#exception---stories-and-tests)
- [React](#react)
  * [Use `e = React.createElement`](#use--e---reactcreateelement-)
  * [Use `native` tables, and very rarely do anything interesting with them](#use--native--tables--and-very-rarely-do-anything-interesting-with-them)
- [Unorganized TODOs](#unorganized-todos)

## This document

This is a document that details the often nuanced ways in which I have developed to write code for Luau, and specifically Roblox. This is not intended to be applicable as a style guide for codebases other than my own, but moreso to implant ideas into your head about what code *can* be.

All guidelines have an explanation as to their intent and will also attempt to show legitimate examples of what caused them to be written. **The goal of this document is for you to agree, not to simply comply.** Guidelines will provide alternatives to patterns where possible so that you are not left in the dark.

At the same time, this document is not interested in topics that are excessively nitpicky, as while I have my opinions on topics such as indentation and casing, so do you and everyone else, and it frankly doesn't matter. To try and alleviate this, every guideline will also try to make obvious how important they actually are. Some guidelines are not opinionated, and are there to prevent guaranteed incorrect behavior. Others are there to make your code simpler, but can be ignored at a cost of some kind, such as complexity, safety, performance, etc. Some guidelines have reasonable exceptions that will be detailed when possible.

## General philosophy

### Code should be strictly typed

All code you write should use Luau types and strict mode. You can enable strict mode in your entire codebase with `.luaurc` containing `{ "languageMode": "strict" }`. If your codebase is already large enough that you cannot easily switch it to strict mode, enable strict language mode anyway and add `--!nonstrict` to the beginning of every existing script. Migrate your codebase over time. Do not use `--!nonstrict` or `--!nocheck` in any new scripts.

### Bugs that can be caught statically should

Sometimes an API only allows a bug to be caught at runtime, if at all. This is often a fault in the API itself, and better use of the language, whether that be [proper typing](#code-should-be-strictly-typed) and/or smarter abstractions, will help prevent more issues.

As a trivial example, the first function is strictly worse than the second one:

```lua
local function bad(x)
	assert(typeof(x) == "number", "I wanted a number!")
	return x + 10
end

local function good(x: number)
	return x + 10
end
```

...because in a properly typed codebase, you should not be able to pass anything other than a number into `good`. In rare cases

### Code should be simple

Avoid overly clever code; code is not your artist's canvas, it is the rote process to build that canvas. Small functions that express obvious intent and behavior will create a more predictable codebase.

### Prefer immutability
Direct mutation of state makes code that uses that state less predictable, especially when yielding is involved.

<!-- TODO: Example? -->

If you are using [React](#react), immutability is critical--if you mutate in the wrong places in React, your code will break.

### Build code that works with tools

Tools such as [linters](https://github.com/Kampfkarren/selene), [stylizers](https://github.com/JohnnyMorganz/StyLua), and even [the type checker](#code-should-be-strictly-typed) live in a yin-yang relationship with the developers that use them. On the one hand, they are born from the guidelines and expectations we have as humans, and were all designed for use on existing codebases built without the knowledge they would be coming. On the other, they are docile and fragile creatures, and every so often you can become more productive by adjusting the way you create code to satisfy their quirks.

### Developer experience takes priority over performance

Often times, clean code will happen to be the fastest way to write something. Great! Other times though, some uglier code is marginally faster.

Unless you can measure a legitimate performance hit, prefer the code that is most readable and closest to the way you think about the code in your head. If your code is [too nuanced or otherwise hard to understand](#code-should-be-simple), you in fact make it more likely to create bugs or performance issues in the future.

When you do notice performance problems that necessitate uglier or more complicated code, try to keep the blast radius small by mangling as little as you can to get your desired performance, as per [It can be okay for implementations to be messier if it improves consumer usage](#it-can-be-okay-for-implementations-to-be-miessier-if-it-improves-consumer-usage).

## General code guidelines

Unlike general philosophies, these are easily understood guidelines you can follow. They are not really Luau specific and you would likely benefit from practicing these in other languages.

### Early return/continue when the function logically should not keep going

Early returning is turning this:

```lua
if x then
	if y then
		if z then
			-- do something
		end
	end
end
```

...into this.

```lua
if not x then
	return
end

if not y then
	return
end

if not z then
	return
end

-- do something
```

You should prefer early returning when the function/loop logically cannot keep going if the condition fails. As an example...this code:

```lua
local function dealDamage(humanoid: Humanoid, damage: number)
	if damage > 0 then
		humanoid.Health -= damage
	end
end
```

...is worse than:

```lua
local function dealDamage(humanoid: Humanoid, damage: number)
	if damage <= 0 then
		return
	end
	
	humanoid.Health -= damage
end
```

...because it is very unlikely that a "deal damage" function will have some behavior when there's no damage being dealt. You only open up the possibility that you add some code after the check and it runs when you don't expect it to.

On the other hand, here is a function that logically can continue if the condition fails:

```lua
local function performVictoryEffects(player, rewards)
	if rewards.cash > 0 then
		moneyExplosion(player)
	end
	
	if rewards.quality == "legendary" then
		playSoundEffect(player, WOAH_SOUND_EFFECT)
	end
end
```

This *could* be rewritten as:

```lua
local function performVictoryEffects(player, rewards)
	if rewards.cash > 0 then
		moneyExplosion(player)
	end
	
	if rewards.quality ~= "legendary" then
		return
	end

	playSoundEffect(player, WOAH_SOUND_EFFECT)
end
```

...but this is worse because it's very easy for us to imagine wanting more effects in here. We may add it on after the sound effect, and come to realize they don't run unless the quality is also legendary, which is irrelevant.


### It can be okay for implementations to be messier if it improves consumer usage

If code doesn't work, or is slow, or whatever, then it is okay for the guts of an implementation to be nasty within reason, as long as consumers do not bear the burden. For example, we could write a function that takes a slice of a table like this:

```lua
local function slice<T>(items: { T }, min: number, max: number): { T }
	local result = {}
	
	for index = min, max do
		table.insert(result, items[index])
	end
	
	return result
end
```

However, this is slower (and potentially buggier, because we do not check index bounds) than `table.move`. At the same time, `table.move`s syntax is a little clunky, so we still may really prefer a `slice` function as a utility. In this case, it is okay for our implementation to be a little less obvious if it means that the rest of our `slice` usages are fast and safe.

```lua
local function slice<T>(items: { T }, min: number, max: number): { T }
	-- I can never really remember which is which, and they're all numbers, but that's okay,
	-- because the rest of my code can just be `slice`.
	return table.move(items, min, max, 1, {})
end
```

### In modules, make a named variable and return it, rather than returning directly

These two will do the same thing:

```lua
-- perform.luau
local function perform()
	-- code
end

return perform

-- Another perform.luau
return function()
	-- code
end
```

Prefer the former, as it makes it easier to Ctrl-Shift-F (Ctrl-T is often slow, and cannot be done on some interfaces like GitHub or `rg`), and also makes reading the code of the module itself easier, as you do not need to know what file you are in.

#### Exception - Stories and specs

Hoarcekat stories return a function of their destructor. `spec` files in Jest 2/TestEZ (but not Jest 3, which is what I actually use and prefer) return a function to run the tests. You will always be Ctrl+P'ing to these files, since they aren't externally required by anything anyway, so there's no point in adding this noise.

### Prefer individual files for libraries and not huge utility libraries

Instead of a large "TableUtil" ("util" being what people who are too cowardly to write "stuff" say) with a bunch of disconnected functions inside of it, split these out into their own modules.

```lua
-- Bad
-- TableUtil.luau
local TableUtil = {}

function TableUtil.flatten()
	-- etc
end

function TableUtil.reverse()
	-- etc
end

return TableUtil
```

```lua
-- Good

-- flatten.luau
local function flatten()
	-- etc
end

return flatten

-- reverse.luau
local function reverse()
	-- etc
end

return reverse
```

Putting these in their own folder is fine, but by splitting them out into their own files you make it easier to work with them, easier to create smaller tests for, easier to share across other codebases, reduce the chance of cyclical dependencies, and you get autocomplete when you just type the functon name. That is, typing "reverse" with Luau LSP will automatically begin to require it.

### Comments should describe now or the future, but not yesterday

Let's assume we have the following function:

```lua
local function dealDamage(victim: Player, damage: number)
	-- code
end

dealDamage(victim, 100)
```

We realize that 100 is too much damage, so we change it:

```lua
dealDamage(victim, 50) -- 100 is too much
```

The comment here is what I call "historical comments". It really only makes sense if you are reading the diff on GitHub, or otherwise already know the history. If you are a new hire, or need to do something else in this function, the comment is complete noise. In this case, the best option is to remove the comment entirely, but sometimes this can come up in the form of rewriting comments, such as:

```lua
-- Used to deal damage as a number, but then we wanted to include damage types.
-- Use DamageInfo instead, you can get one from a number by using `createDamageInfo`.
local function dealDamage(victim: Player, damageInfo: DamageInfo)
```

There's so much noise here for what the programmer actually wants, which is to know how to get DamageInfo from a number (assuming that isn't already a common pattern in the codebase). In this case, I would rewrite the comment as simply:

```lua
-- You can create a DamageInfo from just a number by using `createDamageInfo`
local function dealDamage(victim: Player, damageInfo: DamageInfo)
```

For similar reasons, do not include changelogs at the top of your code, like this:

```
-- Coded by @SuperAwesomeDev
-- 2023-05-01: Fixed a bug where the getCurrentDayInApril() would crash if it was May
-- 2023-04-29: Added new function for getting the current day in April
```

git is your history.

Also, no commented code.

```lua
dealDamage(victim, 100)
-- dealDamage(victim, 50)
```

This is the same thing as our previous comment, except uglier.

### Suffix yielding functions with `Async`

Random yields can cause surprise bugs, especially in contexts like React which do not expect it at all and will completely shatter if you do. Roblox has a pattern of suffixing methods with "Async" to indicate they will yield (whether or not that is on its own confusing is too late to be important). Do the same to make it obvious that a function yields.

```lua
local function doThingAsync()
	local data = getDataAsync()
	act(data)
end
```

This is orthogonal to the idea of using promises, but using promises everywhere is itself not a great code pattern because promises have nearly unusable types (none in the primary package, everyone makes their own), and are often simply overkill compared to consistent naming patterns and `task.cancel` and friends.

### Shallow, not deep copy

If you [prefer immutability](#prefer-immutability), then you will never need to deep copy, ever. It may be, in that moment, convenient, but it will always be a waste of perf comparatively.

Suppose you have the following table:

```lua
local items = {
    {
        name = "Sword",
        damage = 10,
        durability = 40,
    },
    
    {
        name = "Health Potion",
        color = {
            r = 255,
            g = 0,
            b = 0,
        },
        healthToRestore = 100,
    },
    
    -- etc
}
```

You want to change the durability of the sword immutably. If you deep copied this, meaning you cloned every table and its tables inside recursively, then you would waste time cloning the health potion, and its inner table, etc etc.

What you really want is:

```lua
-- I prefer doing this all inline without temporaries where possible,
-- cleans it up a lot.
items = table.clone(items)
items[1] = table.clone(items[1])
items[1].durability -= 10
```

There is no scenario where this doesn't work whilst code stays immutable.

## Luau

### Avoid dynamic requires

Luau's static typechecking works through `require`, but only if the value being required is static. This means...

```lua
local Library = require(Modules.Library)

return {
    Library = Library,
}
```

...will give Luau the ability to type `Library` properly, giving you autofill and type errors when used incorrectly. On the other hand, this will not:

```lua
local modules = {}

for _, module in Modules:GetChildren() do
    modules[module.Name] = require(module)
end

return modules
```

This will type `modules`' values as `any`. In the case of [strict mode](#code-should-be-strictly-typed), this will even give an error as `require` *expects* static values.

For this reason, **avoid dynamic requires wherever possible**. Doing it immediately erodes developer experience and breaks the principle of [Bugs that can be caught statically should](#bugs-that-can-be-caught-statically-should), which is regularly more important than DRY.

### Prefer to avoid the idea of truthiness and falsiness in favor of explicit checks

In Luau, values are "falsy" if they are `false` or `nil`, and "truthy" otherwise.

This means that in practice, these two pieces of code are the same:

```lua
if x.Parent then

if x.Parent ~= nil then
```

Prefer the latter as it helps better articulate the intent of the code. This is especially true in the case of the inverse:

```lua
if not x.Parent then -- Bad

if x.Parent == nil then -- Good
```

...where the latter reads as more reasonable English.

#### Exception - If expressions

You have to balance this with the readability of inversed else's (`if not x then y else z`), as the accompanying nil check is inverse the truthiness check. I find that having short `== nil` branches for ifs are okay, but that it is less readable with if-expressions. So both of these are acceptable:

```lua
local name = if character then character.Name else "Unknown"
local name = if character == nil then "Unknown" else character.Name
```

#### Exception - `and` and `or`

The fact that "nil" is falsy makes [Use `and` and `or` for short circuiting and defaults](#use-and-and-or-for-short-circuiting-and-defaults) demonstrably easier, so it is expected to use it in that case.

### Use `and` and `or` for short circuiting and defaults

In Luau, `and` and `or` do not produce booleans. `x and y` means "if x is truthy (false or nil), give back y", and `x or y` means "give x if it's truthy, otherwise y".

For example:

```lua
print(1 and 2) -- 2
print(1 or 2) -- 1
print(nil or "default") -- default
print(nil and "second") -- nil
```

The right hand side of an `and` or an `or` is only executed if necessary.

```lua
local function f()
    print("calculating f...")
    return 100
end

print(true and f()) -- prints "calculating f..." then 100
print(false and f()) -- never prints anything
```

It is encouraged to use these behaviors where reasonable, such as for defaults.

```lua
local function playSound(sound: Sound, volume: number?)
    local soundClone = sound:Clone()
    soundClone.Volume = volume or 0.5 -- Pick a reasonable default volume of 0.5
    soundClone.Parent = SoundService
    soundClone:Play()
end

local function killPlayer(player: Player)
    -- It is even fine to chain
    local humanoid = player
        and player.Character
        and player.Character:FindFirstChild("Humanoid")
    if humanoid == nil then
        return
    end
    
    humanoid.Health = 0
end
```

#### Exception - Don't use `x and y or z`

In older Lua code, you will often see the following pattern:

```lua
local goldAmount = giveDoubleGold and 1000 or 500
```

This acts as a ternary. That is, if `giveDoubleGold` is `true`, we'll give 1,000 gold, or 500 otherwise. This is an emergent behavior of the semantics we just talked about.

If giveDoubleGold is true:

```
giveDoubleGold and 1000 or 500
-----------------------

Step 1. The left side (giveDoubleGold) is truthy, so return the right side (1000).

1000 or 500
-----------

Step 2. The left side (1000) is truthy, so return it.
```

If giveDoubleGold is false:

```
giveDoubleGold and 1000 or 500
-----------------------

Step 1. The left side is falsy, so return it.

false or 500
------------

Step 2. The left side is falsy, so return the right side (500).
```

Got it? Good. Don't use it.

This immediately falls apart where the values in the ternary are themselves falsy. For example...

```lua
-- Don't give any gold on arena
local goldAmount = gamemode == "arena" and nil or 100
```

This reads exactly the same as the former, but because this ternary is only an emergent property of and/or behavior, it will not do what we intended if the gamemode is "arena".

```
gamemode == "arena" and nil or 100
---------------------------

Step 1. The left side is truthy, so return the right side.

nil or 100
----------

Step 2. The left side is falsy, so return the right side.
```

Oops! To counter-act this, Luau has added [if-then-else expressions](https://luau-lang.org/syntax#if-then-else-expressions), which act as a first class ternary.

```lua
local goldAmount = if gamemode == "arena" then nil else 100
```

You should use this pattern exclusively when dealing with ternaries. However, it is overkill in the case of simple defaults. For example...

```lua
local goldAmount = if gamemode.goldToGive then gamemode.goldToGive else 100
```

This should be more tersely written as `gamemode.goldToGive or 100`. It is not only easier to understand what is going on, but it also prevents a possible bug where your truthy expression doesn't match the condition, such as:

```lua
local goldAmount = if gamemode.goldToGive then default.goldToGive else 100
                                               ------- Oops! I meant gamemode
```

### Don't hide builtins

Avoid code that looks like this:

```lua
local insert = table.insert
local max = math.max
local min = math.min
```

This used to be an optimization trick before Luau, but nowadays the standard code of using these inline is regularly either as fast or faster. It also carries some possibility while reading the code that the function being called is not a builtin.

```lua
-- Is `insert` some special function for our own collections?
-- Is `items` a straight forward table, or is it an afformentioned special construct?
-- The answer to both these questions would be "yes" if `insert` was any other function.
insert(items, sword)

-- Significantly more obvious.
table.insert(items, sword)
```

### Don't use string or table call syntax

Luau has the following sugar:

```lua
call "string"
-- ...is equivalent to...
call("string")

call { stuff }
-- ...is equivalent to...
call({ stuff })
```

[Code should be simple](#code-should-be-simple), so avoid both of these. [StyLua](#use-stylua) will correctly automatically format these back to traditional calls. The extent to which they improve readability is, at best, negligible.

### Type publicly exposed functions fully

Luau is very good at inferring types, and will do so for function arguments and their return types:

```lua
-- attack.luau
local function attack(player, item, target)
    local damage = item.attack(target)
    print(`{player} dealt {damage} damage!`)
    return damage
end

return attack

-- Some other file
-- Will probably work!
attack(LocalPlayer, sword, enemy)
```

However, occasionally the types it creates are unwieldy, such as in the above example where "item" is typed as "something that can index `attack` as a function that takes whatever type `target` is, and returns something". This can create APIs that don't [create type errors when they should](#bugs-that-can-be-caught-statically-should), and also makes it harder to quickly understand what an API does.

For this reason, publicly exposed functions should be fully typed. "Publicly exposed" means that it is possible for another script to call this function *directly*, such as because the script returns it.

```lua
local function attack(player: Player, item: Item, target: Attackable): number
    local damage = item.attack(target)
    print(`{player} dealt {damage} damage!`)
    return damage
end
```

Internal functions are okay to leave untyped, though it is also acceptable to fully qualify them anyway.

```lua
-- This is internal, so it is okay not to type `player` and `damage`
local function attackMessage(player, damage)
    print(`{player} dealt {damage} damage!`)
end

-- It would be equally acceptable to have written:
-- local function attackMessage(player: Player, damage: number)

local function attack(player: Player, item: Item, target: Attackable): number
    local damage = item.attack(target)
    attackMessage(player, damage)
    return damage
end

return attack
```

### Aside from functions, avoid trivial types

Aside from [Type publicly exposed functions fully](#type-publicly-exposed-functions-fully), you should *avoid* unnecessary typing unless it either significantly helps readability, the ability to [catch bugs statically](#bugs-that-can-be-caught-statically-should), or because Luau requires or benefits from it for narrowing.

By typing everything, you only make your code noisier and make it harder to refactor in the future.

```lua
-- Bad! What bug could this possibly prevent?
local MAX_HEALTH: number = 100

-- Bad! `item.attack` should return number, making this type pointless.
local damage: number = item.attack(player)
```

### Don't use `pcall`'s shorthand syntax for methods, sometimes use it in other cases

In Luau, `x:y(z)` is really just sugar for `x.y(x, z)`. Combine this with the fact that `pcall` has the following semantics:

```lua
pcall(call, 1, 2, 3)

-- is basically the same as...
pcall(function()
    call(1, 2, 3)
end)
```

...and you may just stumble upon the following pattern:

```lua
-- Destroy the part, but don't error if we can't
pcall(part.Destroy, part)
```

I find these to be far less readable than the [simple code](#code-should-be-simple).

```lua
pcall(function()
    part:Destroy()
end)
```

To ignore the "cleverness" of the shorthand, it simply doesn't read like a good function should--straight forwardly in English. Compare to this other use of shorthand:

```lua
pcall(saveMoney, player, 100)
```

This function reads straight forwardly. Save money, on the player, with 100 (which obviously will be the amount of money). The method shorthand doesn't read the same--Destroy the part...'s part?

On that note, it would still be acceptable to write the above code in the following way:
```lua
pcall(function()
    return saveMoney(player, 100)
end)
```

This is basically identical, and you may prefer the readability of it.

I would caution against pcalling an anonymous function that calls another function with no arguments, however:

```lua
pcall(function()
    return f()
end)
```

This, to me, is basically just noise compared to `pcall(f)`.

### Prefer `{ [K]: V? }` over `{ [K]: V }` where invalid keys are expected to index
In Luau, indexing a table with an unfilled key will give back `nil`. However, the type system will not report it as such, leading to potential type bugs.

```lua
local playerPoints: { [Player]: number } = {}

local function givePoints(player: Player, amount: number)
    -- Typed as number...
    local currentPoints = playerPoints[player]
    
    -- Oops! If the player had no points, this will error!
    currentPoints += amount
end
```

We can have Luau warn us of this if we instead type `playerPoints` with the value as `number?`.

```lua
local playerPoints: { [Player]: number? } = {}
--                                New ^

local function givePointsBad(player: Player, amount: number)
    -- Typed as number?...
    local currentPoints = playerPoints[player]

    -- Type error! Can't `+=` nil
    currentPoints += amount
end

-- Let's try again...
local function givePoints(player: Player, amount: number)
	local currentPoints = playerPoints[player]

	-- Handle the potential of nil..
	if currentPoints == nil then
		playerPoints[player] = amount
	else
		playerPoints[player] += amount
	end
end
```

### `nil` does not mean "nothing".

In Luau, these two functions mean different things:

```lua
local function returnsNothing()
end

local function returnsNil()
	return nil
end
```

The former returns no values, and the latter returns a value--nil. This is not a pedantic difference and has real effects in code.

```lua
print(returnsNothing()) -- Prints a blank line, as `print()` would
print(returnsNil()) -- Prints "nil", as `print(nil)` would
```

This "nothing" value is coloquially referred to as "void" (though it's technically a unit type, as PL folks would say). It rears its ugly head in the case of passing a function off to specifically native functions, such as `string.format`, `print`, etc, and can cause real bugs in practice if you are not careful.

For this reason, you should be explicit about what value you are returning based on the semantics of the function.

```lua
local function doSomething()
	print("I'm a function that performs a side effect, and I have no sensible return value")
	print("Thus, I return nothing.")
end

local function getAmmo(inventory)
	if inventory.selectedWeapon ~= nil and inventory.selectedWeapon.type == "gun" then
		return inventory.selectedWeapon.ammo
	end
	
	-- Don't forget me!
	-- `getAmmo` with no gun selected returns nil to mean "there's nothing here".
	-- If you forget this, you are returning void.
	return nil
end
```

For simple rules to follow, the number of return types should be consistent in all branches. Do not `return x` in one place and `return` in another.

### Use string style enums, and nothing else

There are a hundred ways to write enums in Luau, but only one you should ever use: string literals. This is because it is the only one that [prevents bugs statically](#bugs-that-can-be-caught-statically-should).

```lua
type Color = "red" | "blue" | "green"

local function setColor(color: Color)
	if color == "red" then
		-- red
	elseif color == "blue" then
		-- blue
	elseif color == "green" then
		-- green
	end
end

-- To use:
setColor("red")
```

When using these kinds of enums, you will also benefit from having the following utility function:

```lua
local function exhaustiveMatch(value: never): never
	error(`Unknown value in exhaustive match: {value}`)
end
```

This is a function that can be used to [prevent bugs relating to not properly matching every value](#bugs-that-can-be-caught-statically-should).

```lua
local function setColor(color: Color)
	if color == "red" then
		-- red
	elseif color == "blue" then
		-- blue
	else
		-- TYPE ERROR! You forgot to match "green".
		exhaustiveMatch(color)
	end
end
```

At the moment, **any library** that looks like the following should be categorically avoided:

```lua
local Color = makeEnum({ "red", "green", "blue" })
```

There may be a day where this can create a string union automatically, but today it is impossible. And even if it could, the benefits are not obvious.

The reason people prefer libraries like this are often because of the following behavior:

```lua
setColor(Color.red)
setColor(Color.pink) -- Runtime error! "pink" is not valid
```

However, this is strictly worse than string enums. First of all, there is no way to make the "red" in Color.red autofill, whereas the Luau LSP will properly autofill "red". Second, `Color.pink` is a runtime error, which is strictly worse than a [static type error](#bugs-that-can-be-caught-statically-should).

You can make a similar pattern with string enums...

```lua
local Color = {
	red = "red" :: Color,
	green = "green" :: Color,
	blue = "blue" :: Color,
}
```

...but why bother? Strings already autofill, and [if your code is strictly typed](#code-should-be-strictly-typed), then this will never catch a bug a [simple](#code-should-be-simple) string union doesn't.

### Use generalized iteration

Avoid any of the following code:

```lua
for i, v in pairs(t) do
for i, v in ipairs(t) do
for i, v in next, t do
```

These are necessary in Lua, but not in Luau, which adds generalized iteration:

```lua
for i, v in t do
```

Generalized iteration will go through values starting from 1, like `ipairs`, up until `t[i]` is nil, which will continue in an unspecified order.

The only case where this is not easily replacable is in the case of mixed or holey arrays, in which case `ipairs` may be necessary, but code like this is often very fragile and should be avoided regardless.

### Keep optional arguments clear

The following function accepts either a `boolean`, a function that returns a `boolean`, or `nil`.

```lua
local function useToggleState(default: boolean? | () -> boolean)
```

...however the information that it is optional is in the middle, affixed to `boolean?`. It would be simpler to write the code like this:

```lua
local function useToggleState(default: (boolean | () -> boolean)?)
-- Also acceptable, sometimes more readable depending on the case, but `nil` should be at the end
local function useToggleState(default: boolean | () -> boolean | nil)
```

### Always give `assert` an error message.

`assert` takes a condition, and errors if it fails.

```lua
assert(hasGold)
```

This is similar to a language like Rust, where we can write:

```rs
assert!(has_gold);
```

If `has_gold` is false, then we will get the following error message:

```
thread 'main' panicked at src/main.rs:5:5:
assertion failed: has_gold
```

However, unlike Rust, which clearly outlines the assertion that fails as being `has_gold`, Luau can only provide us a generic error message:

```
Error:2: assertion failed!
stack backtrace:
[C] function assert
code.luau:2
```

This makes it harder to understand what is going on before delving into the code, which still may not be obvious on its own. For this reason, it is recommended to always provide an error message to `assert`.

```lua
assert(hasGold, "Player has no gold")
```

Sometimes, you will need to assert to narrow types in Luau, such as here:

```lua
local function shallowEqual<T>(x: T, y: T)
	if typeof(x) ~= typeof(y) then
		return false
	end
	
	if typeof(x) == "table" then
		assert(typeof(y) == "table") -- It's impossible for this to fail
	end
end
```

In cases such as this, you have three options:
1. Restructure the code to not need the assert. In this case, we could add `and typeof(y) == "table"` to the condition.
2. Give an error message to the assert that explains the problem, in this case "x was a table, but y was not".
3. Give it the error message `"Luau"`. While this will not be particularly helpful in an error, it can be used in the case where the assert only exists for the sake of the type checker, and for errors that a theoretically better type checker would get. I've had Luau engineers look through these asserts before to figure out if the type checker has improved or not in their area :)

### `assert` should only take constant error messages

In Rust, we can get better error messages by creating a format string like so:

```rs
assert!(gold > 10, "Player has {gold} gold");
```

We can do something similar in Luau:

```lua
assert(gold > 10, `Player has {gold} gold`)
```

However, the key difference here is again that in Rust, `assert` is a macro, which means it can have special behaviors that normal functions do not. Specifically, Rust will only do the work necessary on the error message if it has to. The Rust code compiles to this:

```rs
if !(gold > 10) {
	panic!("Player has {gold} gold");
}
```

...but with Luau, `assert` is a function like any other, and it will evaluate the error message every single time. This is a heavy cost for something that we are hoping nobody ever ever has to see! For this reason, it is preferred that the error message in an `assert` be constant. If you need formatting, it is perfectly readable to split it out into a traditional `if`.

```lua
if gold < 10 then
	error(`Player has {gold} gold`)
end
```

### Have a point to asserting `typeof`

Do not use `assert(typeof(x) == "whatever")` as "just in case" Luau types are wrong.

```lua
local function spendGold(player: Player, gold: number)
	assert(typeof(player) == "Instance" and player:IsA("Player"), "player isn't a Player")
	assert(typeof(gold) == "number", "Gold isn't a number")
	
	-- Do the rest...
end
```

The only time these asserts should trip is if your code [isn't properly strictly typed](#code-should-be-strictly-typed).

There *are* real cases where you may want to do this. The first is with user input over an uncontrolled boundary. For example, in RemoteEvents.

```lua
HurtMe.OnServerEvent:Connect(function(player, damage: number)
	assert(typeof(damage) == "number", "Player sent invalid damage")
	
	dealDamage(player, damage)
end)
```

...however by typing `damage` as number in the arguments, you make this code fragile and easier to mess up.

```lua
HurtMe.OnServerEvent:Connect(function(player, damage: number)
	-- Just adding some more code here...
	if damage < 0 then
		error("Player is trying to heal themselves!")
	end
	
	assert(typeof(damage) == "number", "Player sent invalid damage")
	
	dealDamage(player, damage)
end)
```

This code will statically pass, because `damage` is `number`, but it will fail at runtime. For this reason, cases like this should prefer typing the variables explicitly as `unknown`, and let Luau narrowing figure out the rest.

```lua
HurtMe.OnServerEvent:Connect(function(player, damage: unknown)
	-- This code now statically errors, because you can't compare unknown with a number
	if damage < 0 then
		error("Player is trying to heal themselves!")
	end
	
	assert(typeof(damage) == "number", "Player sent invalid damage")
	
	-- damage is now narrowed to `number`.
	dealDamage(player, damage)
end)
```

You are also free to use your own judgment if your code *is* strictly typed, but you've been bitten by invalid values still somehow creeping in, such as in the case of [array or dictionary indexes](#Prefer--K-V--over--K-V--where-invalid-keys-are-expected-to-index).

### Avoid metatables

[Metatables are complicated](#code-should-be-simple), and you probably shouldn't use them without a good reason.

The most obvious case of metatable usage is with `__index` and classes. There is currently no way to type objects in Luau that does not sacrifice some important code quality metric, like type safety, surface area, or general DX. I find it far simpler to create C-style functions. In [My Movie](https://www.roblox.com/games/13600218266/My-Movie), we have core table types such as `Video`, and then functions that act on it. This looks roughy like:

```lua
type Slide = {
	length: number,
	-- assets...
}

type Video = {
	slides: { Slide },
}
```

...with a separate function for:

```lua
-- videoLength.luau
local function videoLength(video: Video.Video): number
	local total = 0
	
	for _, slide in video.slides do
		total += slide.length
	end
	
	return total
end
```

This is great because it avoids `Video` having an enormous API surface (*lots* of things do something on Video), it always works with types, and it's very simple.

#### You probably don't agree with that, but let's at least agree on...

Ignoring `__index` (and, oh what the heck, `__tostring`), avoid basically every other metamethod. No `__call`, no `__add`, none of that! They're basically always unexpected and confusing, and also do not show up in any autocompletes.

#### Exception - Weak tables

The only way to make a weak table is the `__mode` metamethod. Weak tables are extremely spooky and I would be surprised if you needed more than 1 every 100,000 lines of code, but if you must, it is fine.

### Sort requires alphabetically, and do not section them
You may be familiar with the following structure:

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local React = require(ReplicatedStorage.Packages.React)

local MyComponent = require(ReplicatedStorage.Ui.MyComponent)
local MyComponent2 = require(ReplicatedStorage.Ui.MyComponent2)

local useStuff = require(ReplicatedStorage.Ui.Hooks.useStuff)

local InnerComponent = require(script.InnerComponent)
```

It may seem intuitive that having these structured groups would make for easier to work with code, but in fact it is the opposite. First of all, once there's more than you (or even just you) writing the code, you must make sure you agree on a specific standard of organizing. I have never really seen this properly enforced.

Second, this is code you **never read**. You scroll past this every time--all that whitespace is just speed bumps before you get to your code. The only time you have to touch this code is when you are adding onto it or removing something you used to use.

Following our principle of [Build code that works with tools](#Build-code-that-works-with-tools), we can use Luau LSP's native auto-require behavior mixed with [StyLua's](#Use-StyLua) `sort_requires` configuration to **never have to touch this require block to add requires __ever again__**. You can just type your function name/component, press tab, and it automatically requires it and sorts it alphabetically. This would create the following code:

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local InnerComponent = require(script.InnerComponent)
local MyComponent = require(ReplicatedStorage.Ui.MyComponent)
local MyComponent2 = require(ReplicatedStorage.Ui.MyComponent2)
local React = require(ReplicatedStorage.Packages.React)
local useStuff = require(ReplicatedStorage.Ui.Hooks.useStuff)
```

I cannot stress enough that the benefits of auto-require from Luau LSP dwarf any benefit from sectioning these by probably a factor of a gajillion (scientist approved statistic).

Use the following `stylua.toml` to enforce this:

```toml
[sort_requires]
enabled = true
```


## Roblox

### `GetService` everything

At the top of your file, include your services in alphabetical order through GetService. Luau LSP can do this for you.

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local Workspace = game:GetService("Workspace")
```

Do not use `game.ServiceName`. Not all services are available at script runtime by default, and not all services share the name of their service. RunService is named "Run Service", for example.

Do not use `GetService` after the initial header. It's simply noise.

Do not use the `workspace` global. Workspace is not spectacularly more interesting than any other service--rendering in the 3D world and rendering in the 2D world are both of critical importance, but only the former has a global because it was a decision made in the beginning of Roblox. You would probably be weirded out if something like Lighting was a global.

### Use `UDim2.fromOffset` and `UDim2.fromScale`

Put simply, `UDim2.fromOffset(x, y)` is the same as `UDim2.new(0, x, 0, y)`, and `UDim2.fromScale(x, y)` is the same as `UDim2.new(x, 0, y, 0)`. Prefer these to UDim2.new when possible, as it simply is less noisy, and it's hard to tell at a glance which form you're using with UDim2.new since you're already used to there being zeroes in there.

### Put scripts inside scripts as implementation details

When splitting up the implementation of a script without changing the consumer behavior, put implementation details inside the script. For example, if you have a file like this:

```lua
-- src/shared/Ui/Toolbar.luau
local function ToolbarButton()
	-- etc
end

local function ToolbarRow()
	-- etc
end

local function Toolbar()
	-- etc
end

return Toolbar
```

...you can choose to split it up into the following:

```
src/shared/Ui/Toolbar
L init.luau
L ToolbarButton.luau
L ToolbarRow.luau
```

### Use absolute paths, avoid `script.Parent` outside of implementations, and avoid going up more than one Parent

Prefer this:

```lua
-- ReplicatedStorage/Libraries/something.luau
local library1 = require(ReplicatedStorage.Libraries.library1)
```

...to this:

```lua
local library1 = require(script.Parent.library1)
```

Using absolute paths makes it easier to move a script around later and also do not expose the implementation detail of a script's position in the hierarchy. 

The actual root of the absolute path can be either some service, or a `FindFirstAncestor call`.

```lua
-- Because we are using FindFirstAncestor, moving this script around is much easier
local Plugin = script:FindFirstAncestor("MyPlugin")

local PluginButton = require(Plugin.Components.PluginButton)
```

Using `script` from the main script and `script.Parent` from subscripts is fine when you are following [Put scripts inside scripts as implementation details](#Put-scripts-inside-scripts-as-implementation-details), but avoid going up more than one Parent.

#### Exception - Stories and tests

Stories and tests are companions to one file in specific and should always be next to that script. Thus, if you move one, you will always have to move the other. Stories and tests following this pattern should *always* use `script.Parent` to refer to their relevant file.

## React

These guidelines are intended to be specific to React. You may be able to reuse some of them for other libraries like Fusion, but your mileage will vary.

### Use `e = React.createElement`

We don't have JSX, but we have the next best thing. The community has standardized amongst itself `local e = React.createElement` as the idiom of choice for React components. It drastically improves readability when you can more easily just see the components being created without the noise of `React.createElement`.

### Use `native` tables, and very rarely do anything interesting with them

Write components such that passing native properties looks like this:

```lua
e(Pane, {
	padding = 5, -- Some custom property that doesn't exist on the native Roblox class. Make it a normal property.
	
	native = {
		BackgroundColor3 = Color3.new(1, 1, 1), -- For everything else...
	}
})
```

Avoid doing anything particularly interesting with `native`, but it's okay to do stuff like default BackgroundTransparency to 1 unless a color is specified.

---

(Adapted from [Things I learned using React](https://blog.boyned.com/articles/things-i-learned-using-react/))

It is very common to wrap basic Roblox instances in a component for the sake of easier styling or other utilities. I have a `Pane` component, for instance. Let's create one that looks like this.

```lua
local function Pane(props: {
	children: React.ReactNode,
})
	return e("Frame", {
		-- Some nice defaults
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Size = UDim2.fromScale(1, 1),
	}, props.children)
end
```

However, we of course want the ability to write in our own properties. Traditionally, people like to do this by extending the properties itself, such that the following code will work:

```lua
e(Pane, {
	Position = UDim2.fromScale(0.5, 0.5),
})
```

This would have to be implemented to look like:

```lua
local function Pane(props: {
	children: React.ReactNode,

	[any]: any,
})
	local native = table.clone(props)
	-- Remove any extra fields
	native.children = nil

	return e("Frame", join({
		-- Some nice defaults
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Size = UDim2.fromScale(1, 1),
	}, props), props.children)
end
```

However, I really dislike this approach for a few reasons. One is that every time we add a new property, we must now keep that list of omitted properties up to date--for example, the `Pane` component in My Movie has several utilities on top of it for automatically creating layouts, setting aspect ratios, etc. Second, it means that invalid properties will now *definitely* get through Luau. Third, it means that if Roblox ever adds a property named the same as yours, you now have problems as you try to force it into your component.

For these reasons, I choose to have a `native` property instead.

```lua
local function Pane(props: {
	native: { [any]: any }?,
	children: React.ReactNode,
})
	return e("Frame", join({
		-- Some nice defaults
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Size = UDim2.fromScale(1, 1),
	}, props.native), props.children)
end
```

...which would then be used as:

```lua
return e(Pane, {
	native = {
		Position = UDim2.fromScale(0.5, 0.5),
	}
})
```

## Unorganized TODOs
- Return collections if you are creating new ones, return nothing if you are mutating them
- Immutable functions should return the same object when feasible (not O(n)). `list == list2` passing is awesome
- Shallow, not deep copy
- Use `and` and `or` for React components (link back to general Luau advice)
- Use createNextOrder()
- React contexts: typing patterns, when to use createUnimplemented vs empty function
- Be cautious about multiple separate React states due to concurrent rendering, versus one combined one. Be cautious about multiple set states in a row.
- Use state setter callbacks sometimes.
- Use refs for state that shouldn't re-render.
- Use refs for keeping identity of callbacks (useRefToState).
- Don't store elements for a longer life than render.
- Use withXXX callback wrappers for tests that should do cleanup if they fail
- All React children must have keys
- Don't use Rodux
- Don't use createRef in function components, usually
- Use typed identity functions instead of `::` when there's no easier way to force a type
- Don't tween inside React components
	- For that matter, treat refs as mutable in general
- Don't default to bindings--only use them when you have a material performance improvement from them, which is exceedingly rare in React (vs. Roact).
- Use StyLua. It is correct by definition...except when it's not.
- Never use forwardRef
