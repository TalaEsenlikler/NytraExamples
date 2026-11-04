# Nytra Core Essentials

A playable Blueprint reference room. The first two stations demonstrate real
state changes driven by Nytra Core graphs, without Quest or a character framework.

## Play

In an Unreal Engine 5.8 project with Nytra Core enabled, open **Tools > Nytra
Examples**, choose **Nytra Core Essentials**, then **Add to Project**. Use **Open
Example** to open `Maps/L_CoreEssentials` and press Play. Quest, GASP, and the
developer automation plugin are not required. The map supplies its own GameMode;
you do not need to replace your project's default GameMode or input mappings.

The meshless Character uses the
Blueprint's movement and look inputs. Aim at a control and left-click within five
metres. R reloads the room to reset it; 5 saves and 9 loads the saved state.
Use WASD to move and the mouse to look. Click the game viewport to capture input.

**01 Power Routing:** try Open Gate with power off. Toggle Power, then open the
gate. Toggle Power again to close it. The graph owns the `Powered` Boolean;
Set Parameter writes it and Branch reads it through a binding. A Blueprint Task
animates the mechanism. Switching power off cancels opening before closing.

**02 Cargo Lift:** choose Floor 0 or Floor 1, then Move Lift. The graph stores
`SelectedFloor`, `Destination`, and `Moving`. Destination is bound into the task;
the selected floor is bound into a reusable display node. Controls ignore requests
during movement. Completion is emitted when the platform reaches its destination.

## Ownership

| Blueprint | Responsibility |
| --- | --- |
| BP_EssentialsPawn | Meshless Character movement and look inputs. |
| BP_EssentialsController | Camera trace, interaction message, reset, save/load input. |
| BP_EssentialsHUD | Crosshair and controls. |
| BPI_NytraEssentialsInteractable | Small gameplay interaction contract. |
| BP_StationButton | Configurable signal sender for a station; no progression rules. |
| BP_GraphStation | Owns its Core component, mechanism, display, and station snapshot. |
| BP_StationSchema | Blueprint-defined Station context, resolved from the component owner. |
| BP_MoveMechanism | Task updates while moving, completes on arrival; Core owns timer cleanup. |
| BP_SetStationDisplay / BP_ShowSelectedFloor | Blueprint adapters for text display. |

There is no separate level coordinator. Each station is an independently owned
mechanism with its own graph. Input remains on the player controller. Direct actor
references on the station buttons are editable, local relationships, not global
searches or hardcoded names.

## Rebuild and adapt

Everything shipped at runtime is an ordinary Blueprint or Core asset. Create node
Blueprints from Core Node or Task, configure their Class Defaults, and implement
Execute Input or task events. Create variables in the graph's Variables panel and
drag them into the graph to create Set Parameter nodes. Use the Details binding
menu for Station components and graph variables. Sequence starts signal listeners;
unlimited listeners use Received for repeated interaction.

Replace the simple player interaction with your own input system by sending the
same interface message. Replace visual components or the movement task without
rewriting the variable and branching examples. This sample is local single-player;
it does not implement multiplayer interaction authority.

## Save and load

The `Save` folder contains a Blueprint SaveGame class and a station-state struct.
`SaveEssentials` and `LoadEssentials` on the controller return success and a failure
reason, so a menu can call the same functions as 5/9. The slot name and user index
are configurable on the controller. This small example uses Unreal's synchronous
Save Game to Slot / Load Game from Slot nodes.

Each station has a unique, non-empty `StationId`, assigned in the level and retained
across sessions. `CaptureStationState` combines Core's execution snapshot (graph
variables, active tasks, and signal listeners) with the mechanism's relative
transform and display text. The save also stores the player's transform and
controller rotation. Each movement node's **Snap to Target on Restore** checkbox
chooses whether to resume the saved movement or immediately reach its destination.

To compare the two behaviors, start a mechanism moving, press 5 mid-movement,
let it finish, then press 9. With snapping off it returns to the saved position
and continues; with snapping on it immediately reaches the destination. Try
ending Play, starting again, and pressing 9 to load across sessions. Make a new
save after changing graph configuration.

Loading validates the format, level, station identities, graph assets, and compiled
graph hashes before applying state. The station calls **Capture State** and **Restore State** directly on its existing
Core component. Restore validates compatibility before replacing the current run,
initializes the replacement, and restores ticking without replaying the graph's
Start route. The station keeps its identity/asset checks and gameplay restoration.
Compatibility rejection leaves the current run intact; a later resource/resume
failure leaves the component stopped and reports a reason.
If applying a snapshot fails, the controller attempts to restore a snapshot of the
room taken immediately before loading. Failed loads report a reason. Resetting the
room does not delete the slot. Graph edits can invalidate an older save: create a
new save after changing graph behavior.

The display adapters declare `Restore Support = Stateless` in Class Defaults
because they complete immediately and retain no execution state. The movement task
uses Core's task snapshot support. Its Destination and Speed variables have the
Blueprint **Save Game** flag enabled, retaining the values captured for the active
trip. It does not save timer handles or live component references. Context bindings
resolve those references against the current station after restoration.

Restore the mechanism transform and display text **before** calling Core's
**Restore State**. Task restore callbacks then run against the restored world and
can intentionally override that pose. For example, `On Restore Task` can set the
component's relative location to Destination to finish an opening animation
immediately. Applying the saved transform afterward would overwrite that choice.
Returning true from the callback reports successful restoration; the movement
task's next update detects arrival and completes the task.
Both the snap and non-snap branches must return true. A false return means the
task could not restore, and causes the load to fail with `Resume Failed`.

This is a local single-player reference implementation, not an arbitrary-world save
system. New persistent gameplay state must be added explicitly to the example's
save record and capture/restore functions. No Quest or developer automation module
is required by the shipped Blueprints.

## Visual content

The dark gray grid material, its dependencies, and lighting component settings
are copied from the project's Game Animation Sample LevelPrototyping content.
The example contains its own copies and no LevelVisuals decal, level styling
helpers, GASP locomotion, or runtime reference back to the source project paths.
Epic sample assets remain subject to their original license; keep their provenance
when preparing a public example distribution.

## Distribution

Download this example through **Tools → Nytra Examples**. The catalog and cover
are hosted in NytraExamples; the content ZIP is a GitHub Release attachment, not
part of Git history or the NytraCore plugin. Guides remain outside the ZIP.
Installer receipts are created by installation, never inferred from an arbitrary folder.
