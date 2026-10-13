Coding Challenge: Shipment Exception TrackerProblem ContextYou are working on a logistics tracking system that processes real-time shipment events across distribution hubs. Your task is to build an ExceptionTracker that inspects incoming events, cross-references them with shipment and hub data from an external API, and flags business rule violations (exceptions).Technical Specifications1. Provided API Client & HelperAn API client (ApiClient) and a timestamp conversion utility (to_datetime) are already available in the codebase.ApiClient.get_shipment(shipment_id: str) -> dictReturns data about a shipment:JSON{
  "shipment_id": "SHIP-001",
  "shipment_type": "expedite",      // e.g., "expedite" or "standard"
  "ship_delay_threshold": 30,       // minutes (integer)
  "ship_ts": "2026-06-01T10:00:00Z" // ISO timestamp string
}
ApiClient.get_hub(hub_id: str) -> dictReturns data about a hub:JSON{
  "hub_id": "HUB-A",
  "hub_name": "Austin Distribution Center"
}
to_datetime(ts: str) -> datetimeConverts an ISO timestamp string (e.g., "2026-06-01T10:00:00Z") into a timezone-aware UTC datetime object.Rules & RequirementsImplement ExceptionTracker.process_event(event: dict) -> list[str] according to the following rules:Part 1: Event Processing & Exception RulesFrom each incoming event dictionary, extract event_id, shipment_id, hub_id, event_type, and ts.Expedite Delay (expedite_delay):Add "expedite_delay" if event_type == "delay" and the shipment's shipment_type == "expedite".Shipment Delay (ship_delay):Add "ship_delay" if:$$\text{event\_ts} > \text{shipment\_ts} + \text{timedelta}(\text{minutes}=\text{ship\_delay\_threshold})$$Part 2: Deduplication, Tracking & OptimizationDuplicate Event Detection:If an event_id has already been processed by process_event, immediately return [].Do not invoke any API calls for duplicate events.Repeated Shipment-Hub Delay (repeated_ship_delay):Track whether a ship_delay exception has previously been triggered for a specific (shipment_id, hub_id) pair.If the current event triggers a ship_delay and (shipment_id, hub_id) has previously triggered ship_delay, also append "repeated_ship_delay" to the returned exceptions list.Hub Cache (Latency Optimization):Fetching hub data via get_hub(hub_id) is an expensive operation.Implement an in-memory cache for hub responses keyed by hub_id so that repeated calls for the same hub do not hit the API server.Starter CodePythonfrom datetime import datetime, timezone, timedelta

class ApiClient:
    def get_shipment(self, shipment_id: str) -> dict:
        pass

    def get_hub(self, hub_id: str) -> dict:
        pass

class ExceptionTracker:
    def __init__(self, api_client: ApiClient):
        self.api_client = api_client

    def process_event(self, event: dict) -> list[str]:
        """Process shipment event and return list of exception IDs."""
        pass

def to_datetime(ts: str) -> datetime:
    if ts.endswith("Z"):
        ts = ts[:-1]
    return datetime.fromisoformat(ts).replace(tzinfo=timezone.utc)
Example Scenarios for TestingAssume the API returns the following data:SHIP-001: shipment_type="expedite", ship_delay_threshold=30, ship_ts="2026-06-01T10:00:00Z"HUB-A: hub_id="HUB-A"#Incoming Event InputExpected Return ValueReason / Explanation1{"event_id": "EVT-1", "shipment_id": "SHIP-001", "hub_id": "HUB-A", "event_type": "delay", "ts": "2026-06-01T10:15:00Z"}["expedite_delay"]event_type is "delay" and shipment is "expedite". Event is within the 30-minute threshold, so no ship_delay.2{"event_id": "EVT-2", "shipment_id": "SHIP-001", "hub_id": "HUB-A", "event_type": "transit", "ts": "2026-06-01T10:45:00Z"}["ship_delay"]Event timestamp (10:45) exceeds 10:00 + 30m threshold. First time (SHIP-001, HUB-A) triggers ship_delay.3{"event_id": "EVT-1", "shipment_id": "SHIP-001", "hub_id": "HUB-A", "event_type": "delay", "ts": "2026-06-01T10:15:00Z"}[]Duplicate event_id (EVT-1). Short-circuits instantly without API calls.4{"event_id": "EVT-3", "shipment_id": "SHIP-001", "hub_id": "HUB-A", "event_type": "transit", "ts": "2026-06-01T11:15:00Z"}["ship_delay", "repeated_ship_delay"]Exceeds threshold again for (SHIP-001, HUB-A), which previously triggered a ship_delay in EVT-2.
