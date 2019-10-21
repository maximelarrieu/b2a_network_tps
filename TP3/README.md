# TP3 : Routage INTER-VLAN + mise en situation

## I. Router-on-a-stick

### Prove me that your setup is actually working

#### think about VLANs, ping, etc.

Pour te prouver, on va tester tous les pings en corrélation avec le tableau :

✅ = peuvent se joindre
❌ = ne peuvent pas se joindre

Réseaux | `net1` |  `net2` |  `net3` |  `netP`
--- | --- | --- | --- | ---
 `net1` | ✅ | ❌ | ❌ | ✅
 `net2` | ❌ | ✅ | ❌ | ✅
 `net3` | ❌ | ❌ | ✅ | ✅
 `netP` | ✅ | ✅ | ✅ | ✅

## II. Cas concret