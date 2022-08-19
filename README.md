#huff lending

## call data encoding
### packing calldata
calldata in general shouldn't be packed but some scenarios its very beneficial to pack it

ex: function(param, param2), if param two is an address, or anything less that 32 bytes(less than 32 by a predictable amount) should be packed

this could also be extended to all addresses and other data types that are less than 32 bytes

this is because 12 0s of calldata (addresses are only 20 bytes) is 48 gas for those unnecessary 0s. and we would need to mask the address on top of that. when packing
addresses all we need to do is shr (shift right) by 0x60 (96 bits) and thats all. it will get rid of everything else on the end (like the zeroes or next params), and also act as a mask.
### function selector
an idea is to sort the calldatas by size and match the function to the size of calldata.

ex: function(uint256, address), that would be 68 bytes of calldata, and we would need to match it to the size of the calldata. if we optimize this by packing the address and removing the function selector, it would only be 52 bytes. and we would match via the size. 

this would save 273 (68 gas per calldata byte * 4 bc function selector is 4 msb, and one gas saved as reading calldatasize is only 2 gas) gas assuming the function selector never contained a 0.

an issue arises when the functions end up having the same size. ex: function1(uint256, uint256), function2(uint256, uint256)

a simple solution that will only cost 3 (4 actually but 3 bc it allows for another saving) gas per function. just add a 0 to the front of the calldata, increasing the calldatasize by 1. that would distinguish each function with the same calldata size. 