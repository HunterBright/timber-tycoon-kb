---
name: navmesh-plus-kinematic-waypoints
description: Hybrid NPC movement — NavMeshAgent for pedestrians (free-form walking), kinematic waypoints for vehicles (road paths). Different agents, different systems.
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: genre/tycoon/patterns
  tags: [unity, navmesh, npc, pedestrians, vehicles, hybrid]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# NavMesh + Kinematic Waypoints Hybrid

## When to use
Games with both foot-traffic NPCs and vehicle NPCs. NavMesh is poor for vehicles (no turning radius model, can't reverse), but excellent for free-form pedestrian pathfinding. Vehicles benefit from deterministic waypoint sequences.

## Steps

**Pedestrian NPCs — NavMeshAgent:**
```csharp
// NPCPedestrian.cs
NavMeshAgent agent;
void WalkToCounter(Transform counter) {
    agent.SetDestination(counter.position);
    // Agent handles obstacle avoidance, path recalculation
}
```
- NavMesh baked from terrain + walkable surfaces (sawmill floor, road, bridge)
- Excludes: road surface (NPCs don't walk in the road), water

**Vehicle NPCs — kinematic waypoint sequence:**
```csharp
// NPCVehicle.cs — drives along pre-defined path
List<Transform> waypoints; // entry road → parking lane → slot
int currentWp = 0;

void Update() {
    if (currentWp >= waypoints.Count) return;
    var target = waypoints[currentWp].position;
    // Pure-pursuit style: steer toward lookahead point
    MoveToward(target);
    if (Vector3.Distance(transform.position, target) < 0.5f) currentWp++;
}
```
- Entry path: spawn point → parking area (pre-placed waypoints along road)
- Final approach: PD controller takes over for parking (see [[npc-parking-pd-controller]])
- Waypoints: `TrafficWaypoints` component in scene with lists for different routes

**Why split rather than unified:**
- NavMesh can't handle vehicle turning radius — cars on NavMesh look like they teleport
- Waypoint paths look natural for roads (fixed lanes, predictable)
- Pedestrians need dynamic avoidance (other NPCs, player, objects) — NavMesh handles this

## Why this works
Each agent type uses its appropriate movement system. Pedestrians on NavMesh avoid each other and obstacles automatically. Vehicles follow deterministic road-path sequences — predictable, debuggable, visually clean.

## Trade-offs
- Waypoint maintenance: adding a new road route = place new waypoints. NavMesh pedestrian areas update automatically (just rebake)
- Car–pedestrian collision: vehicles are on road, pedestrians are off road (NavMesh excludes road) — by design they don't share space. Safeguard: `NavMeshObstacle` on parked vehicles carves NavMesh around them
- Waypoint path debugging: hard to visualize without Scene View gizmos. Add `OnDrawGizmos` to `TrafficWaypoints` to draw path lines

## Variants
- **All NavMesh:** use a Unity NavMesh with car physics simulation — works for simple cases, breaks for multi-lane or reverse parking
- **All waypoints:** pre-script every NPC movement including pedestrians — works for very scripted sequences (parade, cutscene), doesn't scale to dynamic gameplay

See also: [[npc-parking-pd-controller]], [[pipeline-style-npc-spawn]]
