# code library

## parametrize

### pydantic

```python
# rest.py
from pydantic import BaseModel, Field, ConfigDict, ValidationError
from enum import Enum, auto, Flag, StrEnum, verify, UNIQUE, CONTINUOUS
from typing import Optional, Any
from typing_extensions import Annotated
from collections.abc import Callable

Endpoint = Field(pattern=r'^\/api\/v2\/\w+\/\w+$', default=None)
# PartyId = Annotated[str, StringConstraints(min_length=64, max_length=64)]
PartyId = Annotated[str, Field(pattern=r'[a-fA-F0-9]{64}', default=None)]
AssetId = Annotated[str, Field(pattern=r'[a-fA-F0-9]{64}', default=None)]
# PartyId = Annotated[str, StringConstraints(pattern=r'[a-fA-F0-9]{64}']
# PartyId = Annotated[str, Field(min_length=64, max_length=64, pattern=r'[a-fA-F0-9]{64}', validate_default=True)]
BaseModel.model_config = ConfigDict(arbitrary_types_allowed=True, validate_assignment=True)
# from abc import ABC


@verify(UNIQUE)
class AccountType(StrEnum):
    ACCOUNT_TYPE_UNSPECIFIED = auto()  # default value
    ACCOUNT_TYPE_INSURANCE = auto()  # contain insurance pool funds for a market
    ACCOUNT_TYPE_SETTLEMENT = auto()  # exist only during settlement or mark-to-market
    ACCOUNT_TYPE_MARGIN = auto()  # required initial margin is allocated to each market from user's general account. Collateral in the margin account can't be withdrawn or used as margin on another market until it is released back to the general account
    ACCOUNT_TYPE_GENERAL = auto()  # contain the collateral for a party that is not otherwise allocated. A party will have multiple general accounts, one for each asset they want to trade with. General accounts are where funds are initially deposited or withdrawn from, it is also the account where funds are taken to fulfil fees and initial margin requirements
    ACCOUNT_TYPE_FEES_INFRASTRUCTURE = auto()  # contain fees earned by providing infrastructure on Vega
    ACCOUNT_TYPE_FEES_LIQUIDITY = auto()  # contain fees earned by providing liquidity on Vega markets
    ACCOUNT_TYPE_FEES_MAKER = auto()  # hold fees earned by placing orders that sit on the book and are then matched with an incoming order to create a trade - These fees reward parties who provide the best priced liquidity that actually allows trading to take place
    ACCOUNT_TYPE_BOND = auto()  # created to maintain liquidity providers funds commitments
    ACCOUNT_TYPE_EXTERNAL = auto()  # represents an external source (deposit/withdrawal)
    ACCOUNT_TYPE_GLOBAL_INSURANCE = auto()  # global insurance account for the asset
    ACCOUNT_TYPE_GLOBAL_REWARD = auto()  # global reward account for the asset
    ACCOUNT_TYPE_PENDING_TRANSFERS = auto()  # per asset account used to store pending transfers (if any)
    ACCOUNT_TYPE_REWARD_MAKER_PAID_FEES = auto()  # per asset reward account for fees paid to makers
    ACCOUNT_TYPE_REWARD_MAKER_RECEIVED_FEES = auto()  # per asset reward account for fees received by makers
    ACCOUNT_TYPE_REWARD_LP_RECEIVED_FEES = auto()  # per asset reward account for fees received by liquidity providers
    ACCOUNT_TYPE_REWARD_MARKET_PROPOSERS = auto()  # per asset reward account for market proposers when the market goes above some trading threshold
    ACCOUNT_TYPE_HOLDING = auto()  # per asset account for holding in-flight unfilled orders' funds
    ACCOUNT_TYPE_LP_LIQUIDITY_FEES = auto()  # network controlled liquidity provider's account, per market, to hold accrued liquidity fees
    ACCOUNT_TYPE_LIQUIDITY_FEES_BONUS_DISTRIBUTION = auto()  # network controlled liquidity fees bonus distribution account, per market
    ACCOUNT_TYPE_NETWORK_TREASURY = auto()  # network controlled treasury
    ACCOUNT_TYPE_VESTING_REWARDS = auto()  # account holding user's rewards for the vesting period
    ACCOUNT_TYPE_VESTED_REWARDS = auto()  # account holding user's rewards after the vesting period
    ACCOUNT_TYPE_REWARD_AVERAGE_POSITION = auto()  # per asset market reward account given for average position
    ACCOUNT_TYPE_REWARD_RELATIVE_RETURN = auto()  # per asset market reward account given for relative return
    ACCOUNT_TYPE_REWARD_RETURN_VOLATILITY = auto()  # per asset market reward account given for return volatility
    ACCOUNT_TYPE_REWARD_VALIDATOR_RANKING = auto()  # per asset market reward account given to validators by their ranking
    ACCOUNT_TYPE_PENDING_FEE_REFERRAL_REWARD = auto()  # per asset account for pending fee referral reward payouts
    ACCOUNT_TYPE_ORDER_MARGIN = auto()  # per asset market account for party in isolated margin mode


@verify(UNIQUE)
class ProposalState(StrEnum):
    """Proposal states"""
    STATE_UNSPECIFIED = auto()  # Default value, always invalid
    STATE_FAILED = auto()  # Proposal enactment has failed - even though proposal has passed, its execution could not be performed
    STATE_OPEN = auto()  # Proposal is open for voting
    STATE_PASSED = auto()  # Proposal has gained enough support to be executed
    STATE_REJECTED = auto()  # Proposal wasn't accepted (proposal terms failed validation due to wrong configuration or failing to meet network requirements)
    STATE_DECLINED = auto()  # Proposal didn't get enough votes (either failing to gain required participation or majority level)
    STATE_ENACTED = auto()  # Proposal enacted
    STATE_WAITING_FOR_NODE_VOTE = auto()  # Waiting for node validation of the proposal


@verify(UNIQUE)
class ProposalType(StrEnum):
    """Proposal typings"""
    TYPE_UNSPECIFIED = auto()  # Default value, always invalid
    TYPE_ALL = auto()  # List all proposals
    TYPE_NEW_MARKET = auto()  # List new market proposals
    TYPE_UPDATE_MARKET = auto()  # List update market proposals
    TYPE_NETWORK_PARAMETERS = auto()  # List change network parameter proposals
    TYPE_NEW_ASSET = auto()  # New asset proposals
    TYPE_NEW_FREE_FORM = auto()  # for creating n ew free form proposal
    TYPE_UPDATE_ASSET = auto()  # Update asset proposals
    TYPE_NEW_SPOT_MARKET = auto()  # New spot market proposals
    TYPE_UPDATE_SPOT_MARKET = auto()  # Update existing spot market
    TYPE_NEW_TRANSFER = auto()  # New transfer proposal
    TYPE_CANCEL_TRANSFER = auto()  # Cancel transfer
    TYPE_UPDATE_MARKET_STATE = auto()  # Update market state
    TYPE_UPDATE_REFERRAL_PROGRAM = auto()  # Update referral program
    TYPE_UPDATE_VOLUME_DISCOUNT_PROGRAM = auto()  # Update volume discount program


class Pagination(BaseModel):
    first: Optional[int] = 0  # Number of records to be returned that sort greater than row identified by cursor supplied in 'after'
    after: Optional[str] = ""  # If paging forwards, the cursor string for the last row of the previous page
    last: Optional[int] = 0  # Number of records to be returned that sort less than row identified by cursor supplied in 'before'
    before: Optional[str] = ""  # If paging forwards, the cursor string for the first row of the previous page
    newestFirst: bool = True  # Whether to order the results with the newest records first


class AccountFilter(BaseModel):
    # pydantic makes a deepcopy of default value []
    assetId: AssetId = ""  # Restrict accounts to those holding balances in this asset ID
    partyIds: list[PartyId] = []  # Restrict accounts to those with the given party IDs
    marketIds: list[str] = []  # Restrict accounts to those connected to the markets in this list. Pass an empty list for no filter.
    accountTypes: list[AccountType] = []  # Restrict accounts to those connected to any of the types in this list. Pass an empty list for no filter.


class ParamsAccountBalanceChanges(BaseModel):
    model_config = ConfigDict(revalidate_instances='never')
    target: Endpoint = "/api/v2/balance/changes"
    filter: AccountFilter = AccountFilter()
    pagination: Pagination = Pagination()  # returns configurable pagination dict


class ParamsGovernanceAllProposals(BaseModel):
    """Query a list of all proposals"""
    proposalState: ProposalState = ProposalState.STATE_UNSPECIFIED
    proposalType: ProposalType = ProposalType.TYPE_UNSPECIFIED
    proposerPartyId: str = Field(pattern=r'[a-fA-F0-9]{64}',
                                 default=None)  # restrict proposal to those proposed by the given party ID
    proposalReference: str = None  # Restrict proposals to those with the given reference


class RestCall(BaseModel):
    target: Endpoint
    params: Callable[[Any], dict]  # https://docs.python.org/3/library/typing.html#annotating-callables
    usePagination: bool = True  # use pagination in rest api

```

### custom parametrization

parametrization of dictionary values after doing `pydantic.dump_model`

```python
from vegavote.typings.custom import DictSimple, DictSimpleList, BasicTypes, DictItems


def parametrize_basic(parent_key: str, d: DictSimple, sep: str) -> str:
    """ {'first': 5} converts to 'parent_key.first=5&' """
    (key, value), = d.items()  # tuple comma is needed!
    return f"{parent_key}.{key}={value}{sep}"


def parametrize_list(key: str, vals: list[BasicTypes], glue: bool = False, sep: str = "&") -> str:
    if glue:  # nums=1,2,3&
        return f"{key}=" + "".join(f"{v}," if v != vals[-1] else f"{v}{sep}" for v in vals)
    else:  # foreach n in list separate eg: nums=1&nums=2&nums=3&
        return "".join(f"{key}={v}{sep}" for v in vals)


def parametrize_dict(parent_key: str, dictionary: dict, sep: str) -> str:
    built_url = []
    # if value inside the dictionary is another mapping
    dict_inner_value = tuple(dict.values(dictionary))[0]  # dict values obj is not subscriptable so conversion to tuple
    if isinstance(dict_inner_value, list):
        # eg: "filter": {'partyIds': ['d30dd1cffe', '5cb1faaac866']}
        key = tuple(dict.keys(dictionary))[0]
        # splitting because parent_key needs to be added to both list elements, throw away last split
        parametrized_elements = parametrize_list(key=key, vals=dict_inner_value, sep=sep).split(sep)[:-1]
        built_url.append("".join(f"{parent_key}.{x}{sep}" for x in parametrized_elements))
        # built_url.append(parametrize_dict_inner_list(parent_key, dictionary, sep=sep))
    elif isinstance(dict_inner_value, dict):
        # add parent key with dot notation and recursively call this function
        (inner_key, inner_value), = dictionary.items()
        built_url.append(parametrize_dict(parent_key + "." + inner_key, inner_value, sep=sep))
    elif isinstance(dict_inner_value, BasicTypes):
        built_url.append(parametrize_basic(parent_key, dictionary, sep=sep))
    else:
        raise Exception(f"Unhandled datastructure value inside dictionary: {type(dict_inner_value)}")
    return "".join(built_url)


def parametrize_on_type(d: DictItems, sep: str = "&") -> str:
    key, val = d  # tuple comma is needed!
    if isinstance(val, dict):  # validate record is a dictionary
        return parametrize_dict(parent_key=key, dictionary=val, sep=sep)
    elif isinstance(val, BasicTypes):
        return f"{key}={val}{sep}"
    elif isinstance(val, list):
        return parametrize_list(key, val, sep=sep)
    else:
        raise Exception(f"investigate unsupported type: {type(val)}")


def build_params(parameters: dict, sep: str = "&") -> str:
    return "".join(parametrize_on_type(r, sep) for r in parameters.items())

```

testing it

```python
import pytest  # https://docs.pytest.org/en/7.1.x/getting-started.html
from vegavote.typings.custom import DictSimple, DictSimpleList
from vegavote import parametrize as par
from collections import defaultdict


@pytest.mark.parametrize("parent_key, d, sep, expected", [
    ('filter', {'first': 5}, '&', 'filter.first=5&'),
])
def test_parametrize_basic(parent_key: str, d: DictSimple, sep: str, expected: str):
    assert par.parametrize_basic(parent_key, d, sep) == expected


def test_parametrize_list():
    assert par.parametrize_list('some_numbers', [1, 2, 3], glue=True) == 'some_numbers=1,2,3&'
    assert par.parametrize_list('nums', [1, 2, 3], glue=False) == 'nums=1&nums=2&nums=3&'


@pytest.mark.parametrize("parent_key, d_row, sep, expected", [
    ('filter', {'partyIds': ['d30dd1cffe', '5cb1faaac866']}, '&',
     'filter.partyIds=d30dd1cffe&filter.partyIds=5cb1faaac866&'),
    ('recursive', {'filter': {'partyIds': ['lol', 'mao']}}, '&',
     'recursive.filter.partyIds=lol&recursive.filter.partyIds=mao&'),
])
def test_parametrize_dict(parent_key, d_row, sep, expected):
    assert par.parametrize_dict(parent_key, d_row, sep) == expected


@pytest.fixture()
def data_sample_given():
    return [{'filter': {'partyIds': ['d30dd1cffe', '5cb1faaac866']},
             'pagination': {'first': 5},
             "lol": "mao"},
            {'nums': [1, 2, 3],
             'a_string': 'hello',
             'a_dictionary': {'lol': 'mao'},
             'recursive': {'filter': {'partyIds': ['d40fe', 'lolmao']}}
             }]


@pytest.fixture()
def data_sample_expected():
    return [
        'filter.partyIds=d30dd1cffe&filter.partyIds=5cb1faaac866&pagination.first=5&lol=mao&',
        'nums=1&nums=2&nums=3&a_string=hello&a_dictionary.lol=mao&recursive.filter.partyIds=d40fe&recursive.filter.partyIds=lolmao&'
    ]


def test_build_params(data_sample_given, data_sample_expected):
    assert par.build_params(data_sample_given[0]) == data_sample_expected[0]
    assert par.build_params(data_sample_given[1]) == data_sample_expected[1]

```

### main test

```python
import asyncio
import time
import aiohttp
import random
import pydantic
import websockets as ws  # websocket refers to another lib called websocket-client
import json
# import grpc
from pprint import pprint
from vegavote.exceptions import RestAPIRequestError
from vegavote.store import vegastore as vs
from vegavote.typings import rest  # custom Vega typing
from typing import Final
from collections.abc import Callable
from vegavote.parametrize import build_params
import re
import uuid
# async with asyncio.timeout(10):  # python ≥ 3.11

mainnet: Final = False  # False -> use testnet


async def get_headers(url: str):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return response.headers


# Get governance data (proposals and votes) for all proposals
# doing rest api first, switching to grpc afterwards
"""
async def get_grpc_data():
    #  GetProposals(GetProposalsRequest) returns (GetProposalsResponse);
    async with grpc.aio.insecure_channel("localhost:50051") as channel:
        stub = helloworld_pb2_grpc.GreeterStub(channel)
        response = await stub.SayHello(helloworld_pb2.HelloRequest(name="you"))
    print("Greeter client received: " + response.message)
"""


def build_rest_url(endpoint: str, params: dict) -> str:
    base_url = random.choice(vs.rest_node[network]) + endpoint
    parametrized = build_params(params)
    return f"{base_url}?{parametrized}"


async def get_rest_data(url: pydantic.HttpUrl) -> dict:
    # todo: check ratelimit headers
    header = {
        'Accept': 'application/json',
    }
    """ rate limit parts returned in headers response
    "ratelimit-limit": "1200",
    "ratelimit-remaining": "3999",
    """
    api_used = url.split("?")[0]
    async with aiohttp.ClientSession(headers=header) as session:
        # deal with eg 400 error
        try:
            # await asyncio.wait_for(session.get(base_url + endpoint), 10)
            async with session.get(url) as response:
                pprint(response.headers)  # might not work yet
                print("X-Vega-Node-Id: ", response.headers["X-Vega-Node-Id"])
                return await response.json()
        except asyncio.TimeoutError:
            raise RestAPIRequestError(url=url, endpoint=api_used, network=network)
        except aiohttp.ContentTypeError as error:
            print(f"using a Rest API {url} that is unable to return content {error}")


# todo: store as feather, store to db
# doing rest data first, then comes websockets
async def receive_from_websocket(url: str):
    responses_store = []
    async with ws.connect(url) as websocket:
        while True:
            try:
                response = await websocket.recv()
            except ws.ConnectionClosedOK:
                break
            except ws.ConnectionClosedError as err:
                print(f"url is probably invalid: {url}. Got error: {err}")
            responses_store.append(json.loads(response))


async def rest_api_calls(rest_calls: list[pydantic.HttpUrl]):
    endpoint_pattern = re.compile(r"api/v2/(\w+)/(\w+)\?")
    try:
        async with asyncio.TaskGroup() as tg:  # automatic cancellation and exception propagation
            results = []
            for call in rest_calls:
                m = endpoint_pattern.search(call)
                try:
                    name_of_task = f"{m.group(1)}_{m.group(2)}" "-" + str(uuid.uuid4()).split('-')[3]
                except AttributeError:
                    print("no match for your regex on this url: ", call)
                start = time.time()
                task = tg.create_task(
                    asyncio.wait_for(
                        get_rest_data(call),
                        timeout=100)
                )
                task.add_done_callback(lambda t: results.append({f"{name_of_task}": t.result()}))
    except asyncio.TimeoutError:
        print("timeout")
    else:
        pprint(results, indent=2, width=120, compact=True)
        end = time.time()
        print("time taken: ", end - start)
        print("tasks finished")


class RestQueryBuild:
    def __init__(self, target: str, params: Callable, pagination: bool = False):
        self.target = target
        self.params = params
        self.pagination = pagination


def main():
    # await receive_from_websocket(url_vega_gov)
    # running app for mainnet or testnet?
    # todo: make function doing rest api calls with this task group
    accounts_to_check = ["d60d85b145967f53f5714184049a057759a73b0a5757015174a5a652f9715f55",
                         "5990f199811f234c99259b9c48b52a0bf9198107ab01364ff0c151329aaac899"]
    balance_changes = rest.ParamsAccountBalanceChanges()
    balance_changes.filter.partyIds = accounts_to_check
    balance_changes.pagination.first = 5  # retrieve first 5 results
    params = balance_changes.model_dump(exclude_defaults=True)
    balance_changes_url = build_rest_url(endpoint=balance_changes.target, params=params)
    list_of_calls_to_make = [balance_changes_url]  # set url on model?

    """
    response headers: 
    <CIMultiDictProxy('Alt-Svc': 'h3=":443"; ma=2592000', 'Content-Encoding': 'gzip', 'Content-Length': '119',
    'Content-Type': 'application/json', 'Date': 'Sat, 02 Mar 2024 17:43:20 GMT',
    'Ratelimit-Limit': '1200',
    'Ratelimit-Remaining': '3999',
    'Ratelimit-Request-Remote-Addr': '127.0.0.1',
    'Ratelimit-Reset': '1',
    'Server': 'Caddy', 'Vary': 'Origin', 'X-Block-Height': '23919336', 'X-Block-Timestamp': '1709401363657948000',
    'X-Vega-Node-Id': 'n06.test.vega.node')>

    """

    asyncio.run(rest_api_calls(list_of_calls_to_make))



if __name__ == '__main__':
    network = "mainnet" if mainnet else "testnet"
    main()
```
