# Resource-Allocator
from flask import Flask
import copy
import json

app = Flask(__name__)

min_soln = {"cost": float("inf"), "machines": {}}


def util(region_costs, units, capacity, hours, cost, memo = {}):
    global min_soln

    if capacity < 0:
        return float("inf")

    if capacity == 0:
        # print("Reaches here with cost", cost)
        if cost < min_soln["cost"]:
            min_soln["cost"] = cost

        return cost

    if capacity in memo and cost in memo[capacity]:
        return memo[capacity][cost]


    min_cost = float("inf")
    for key, unit in units.items():

        if key not in region_costs:
            continue

        cost_per_region = region_costs[key] * hours

        current_cost = util(region_costs, units, capacity - unit, hours, cost + cost_per_region, memo)

        min_cost = min(min_cost, current_cost)

    if capacity not in memo:
        memo[capacity] = {}
    memo[capacity][cost] = min_cost
    return min_cost


def min_resource(capacity, hour):
    # global min_soln
    region = {
        "New York": {
            "Large": 120,
            "XLarge": 230,
            "2XLarge": 450,
            "4XLarge": 774,
            "8XLarge": 1400,
            "10XLarge": 2820
        },
        "China": {
            "Large": 110,
            "XLarge": 200,
            "4XLarge": 670,
            "8XLarge": 1180
        },
        "India": {
            "Large": 140,
            "2XLarge": 413,
            "4XLarge": 890,
            "8XLarge": 1300,
            "10XLarge": 2970
        }
    }

    units = {
        "10XLarge": 320,
        "8XLarge": 160,
        "4XLarge": 80,
        "2XLarge": 40,
        "XLarge": 20,
        "Large": 10,
    }

    result = []
    for k, r in region.items():
        min_soln["cost"] = float("inf")
        util(r, units, capacity, hour, 0, {})
        result.append({"region": k, "total_cost": copy.copy(min_soln["cost"])})

    return result

@app.route('/resource/<capacity>/<hour>')
def hello_world(capacity, hour):

    return json.dumps(min_resource(int(capacity), int(hour)))



app.run("127.0.0.1", "3000")

