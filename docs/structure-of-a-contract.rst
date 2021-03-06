.. index:: contract, state variable, function, metadata;

.. _contract_structure:

***********************
Structure of a Contract
***********************

Contracts in Vyper are contained within files, with each file being one smart-contract.  Files in Vyper are similar to classes in object-oriented languages.
Each file can contain declarations of :ref:`structure-state-variables`, :ref:`structure-functions`, and :ref:`structure-structs-types`.

.. _structure-state-variables:

State Variables
===============

State variables are values which are permanently stored in contract storage.

::

  storedData: int128

See the :ref:`types` section for valid state variable types and
:ref:`visibility-and-getters` for possible choices for
visibility.

.. _structure-functions:

Functions
=========

Functions are the executable units of code within a contract.

::

  @public
  @payable
  def bid(): // Function
    // ...

:ref:`Function-calls` can happen internally or externally
and have different levels of visibility (:ref:`visibility-and-getters`)
towards other contracts. Functions must be decorated with either @public or @private.

Default function
----------------

A contract can also have a default function, which is executed on a call to the contract if no other functions match the given function identifier (or if none was supplied at all, such as through someone sending it Eth). It is the same construct as fallback functions `in Solidity <https://solidity.readthedocs.io/en/latest/contracts.html?highlight=fallback#fallback-function>`_. 

This function is always named `__default__` and must be annotated with `@public`. It cannot have arguments and cannot return anything.

If the function is annotated as `@payable`, this function is executed whenever the contract is sent Ether (without data). This is why the default function cannot accept arguments and return values - it is a design decision of Ethereum to make no differentiation between sending ether to a contract or a user address.

**Example:**

::

    Payment: event({amount: int128, from: indexed(address)})
    
    @public
    @payable
    def __default__():
        log.Payment(msg.value, msg.sender)


Considerations
~~~~~~~~~~~~~~

Just as in Solidity, Vyper generates a default function if one isn't found, in the form of a REVERT call. Note that this still `generates an exception <https://github.com/ethereum/wiki/wiki/Subtleties>`_ and thus will not succeed in receiving funds. 

Ethereum specifies that the operations will be rolled back if the contract runs out of gas in execution. ``send`` calls to the contract come with a free stipend of 2300 gas, which does not leave much room to perform other operations except basic logging. **However**, if the sender includes a higher gas amount through a ``call`` instead of ``send``, then more complex functionality can be run. 

It is considered a best practice to ensure your payable default function is compatible with this stipend. The following operations will consume more than 2300 gas:

- Writing to storage
- Creating a contract
- Calling an external function which consumes a large amount of gas
- Sending Ether

Lastly, although the default function receives no arguments, it can still access the ``msg`` global, including:

- the address of who is interacting with the contract (``msg.sender``)
- the amount of ETH sent (``msg.value``)
- the gas provided (``msg.gas``).


.. _structure-events:

Events
======

Events may be logged in specially indexed data structures that allow clients, including light clients, to efficiently search for them.

::

    Payment: event({amount: int128, arg2: indexed(address)})

    total_paid: int128

    @public
    @payable
    def pay():
        self.total_paid += msg.value
        log.Payment(msg.value, msg.sender)

Events must be declared before global declarations and function definitions.

.. structure-metedata:

NatSpec Metadata
================

Vyper supports structured documentation for state variables and functions and events.

::

  carrotsEaten: int128
  """
  @author Bob Clampett
  @notice Number of carrots eaten
  @dev Chewing does not count, carrots must pass the throat to be "eaten"
  """

::

  @public
  @payable
  def doesEat(food: string):
    """
    @author Bob Clampett
    @notice Determine if Bugs will accept `food` to eat
    @dev Compares the entire string and does not rely on a hash
    @param food The name of a food to evaluate (in English)
    @return true if Bugs will eat it, false otherwise
    """
  
    // ...

::

  Ate: event({food: string})
  """
  @author Bob Clampett
  @notice Bugs did eat `food`
  @dev Chewing does not count, carrots must pass the throat to be "eaten"
  @param food The name of a food that was eaten (in English)
  """



Additional information about Ethereum Natural Specification (NatSpec) can be found `here <https://github.com/ethereum/wiki/wiki/Ethereum-Natural-Specification-Format>`_. 


Contract Interfaces
===================

Vyper supports exporting and importing contract interfaces, this is done using a `import` and `implements` statements.

::

    import an_interface as FooBarInterface

    implements: FooBarInterface

::


This will import the defined interface in vyper file at `an_interface.vy` and make sure the current contract implements all the necessary public functions.
Note that all interface is valid vyper code, without the return type check. Meaning you can use a contract with code in in the function body as interface as well (but default to a function body with a `pass`).


Extracting Interfaces
---------------------

Vyper has a built-in format option to allow you to make your own vyper interfaces easily.

::

    vyper -f interface examples/voting/ballot.vy

    # Functions

    @constant
    @public
    def delegated(addr: address) -> bool:
        pass

    # ...

::

If you want to do an external call to another contract, vyper provides an external contract extract utility as well.

::

    vyper -f external_interface examples/voting/ballot.vy

    # External Contracts
    contract Ballot:
        def delegated(addr: address) -> bool: constant
        def directlyVoted(addr: address) -> bool: constant
        def giveRightToVote(voter: address): modifying
        def forwardWeight(delegate_with_weight_to_forward: address): modifying
        # ...
::

The output can then easily be copy-pasted to be consumed.

Built-in Interfaces
-------------------
Vyper supports a few built-in interfaces such as ERC20 and ERC721. These are import from `vyper.interfaces`.

::

  from vyper.interfaces import ERC20

  implements: ERC20

::
