# Integration

### async_setup_entry(hass: HomeAssistant, config_entry: ConfigEntry) -> bool

- Retrieves the api token from the config_entry.
- Tries to connect to the API.
- Starts the loop.
- Waits until `CHARGE_POINTS` is received.
- Stores the connector, so it can be retrieved in other classes.
- Tells Ha to start the platform setup.
- Sets disconnect to be called when the integration is unloaded.
- Defines all service methods.
- Registers services.

### async_unload_entry(hass: HomeAssistant, config_entry: ConfigEntry) -> bool

- Unloads the config entry.

### set_entities_unavailable(hass: HomeAssistant, config_id: str)

- Sets all BlueCurrent entities to unavailable.

## Connector

charge_points: dict[str, dict]
grid: dict[str, Any]

### \_\_init\_\_(hass: HomeAssistant, config: ConfigEntry, client: Client)

- Stores the given parameters.

### connect(token: str)

- Sets `on_data` as the receiver.
- Connects to the API.

### on_data(message: dict)

[](../flow/on-data.md)

Routes the received message to the correct function based on the object.

#### handle_charge_points(data: list)

- Adds all evse_ids to `charge_points` and sends status requests for each. Also sends grid status request.

### get_charge_point_data(self, evse_id: str)

- Requests the status and settings of the charge point with the given evse_id.

### add_charge_point(self, evse_id: str, model: str, name: str)

- Adds the charge point to `charge_points`

### update_charge_point(self, evse_id: str, data: dict)

- Updates the key, value pairs of charge point with the given evse_id and dispatches a signal.

#### handle_activity(data: dict)

- The state of the block / operative switch is not returned with `CH_STATUS`, But the state is related to the state of the `activity` sensor. So this function sets the state of `block` switch based on the state of `activity`.

### dispatch_value_update_signal(self, evse_id: str)
- Dispatches a `value_update`

### dispatch_grid_update_signal(self) -> None:
- Dispatches a `grid_update`

### start_loop()

- Tries to start the loop for listening to the API.
- If an error happens, a warning is logged and the reconnect method is called after 1 second.

```{note}
The reason for the one second delay is that the connection closes quite frequently so with this it 'immediately' reconnects.
```

### reconnect(self, event_time: datetime | None = None)

- Tries to reconnect the API, starts the loop again and requests the charge points.
- If the reconnection fails the entities will be set unavailable and the reconnect method will be called again in 20 seconds.
- If the error was `request limit reached`, the reconnect method is called after the next limit reset.

### disconnect()

- Disconnects with the API, if the connection was already closed and a `WebsocketError` is thrown, ignore it.

## BlueCurrentEntity(Entity)

### \_\_init\_\_(connector: Connector, evse_id: str | None = None)

- Stores connector
- If evse_id is given, device info is added to entity.

### async_added_to_hass()

- Connects the dispatcher to a signal to call `update` when triggered.

Update method calls `update_from_latest_data()` and writes the state to HA.

### update_from_latest_data()

- Method that is implemented in the platform classes.
