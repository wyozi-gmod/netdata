# netdata

Reduced-boilerplate networking for data associated with an entity.

This library allows networking complex data (anything net library supports) to all players in PVS automatically. If you need to send long strings or JSON, this library is a good fit. If all you need is simple datatypes, use [normal data tables](https://wiki.facepunch.com/gmod/Networking_Entities).

Additionally, the library has a small utility for sending entity-associated data from client to server.

## Usage

Put `netdata.lua` somewhere and include it both server- and clientside. The library injects itself into the Entity metatable, so you don't need to include it in any specific way.

## Example

**Server -> Client**

```lua
ENT.Type = "anim"

if SERVER then
  function ENT:Initialize()
    self:UpdateEssay()
    timer.Create("EssayUpdater", 60, 0, function()
      self:UpdateEssay()
    end)
  end
  
  function ENT:UpdateEssay()
    self.LongEssay = file.Read("essay.txt", "DATA")
    
    -- Resends data to all players in PVS
    self:NetDataUpdate()
  end
  
  -- Writes the net message we will send to players
  -- This will be called in two situations:
  --   a) when you call NetDataUpdate
  --   b) when player (who hasn't received this data before) enters PVS 
  function ENT:NetDataWrite()
    net.WriteString(self.LongEssay)
  end
end
if CLIENT then
  function ENT:NetDataRead()
    self.Essay = net.ReadString()
    print("Received essay; now draw it or something?")
  end
end
```

**Client -> Server**

```lua
ENT.Type = "anim"

if SERVER then
  function ENT:ReceiveNetAction(cl)
    local str = net.ReadString()
    print(cl, "sent us", str)
  end
end
if CLIENT then
  function ENT:OnButtonPress()
    self:StartNetAction()
    net.WriteString("something something")
    net.SendToServer()
  end
end
```
