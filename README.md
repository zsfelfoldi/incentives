## Some pointers to help understanding the LES incentivization model

### My Devcon5 talk slides:

https://github.com/zsfelfoldi/incentives/blob/master/fzs_slides.pdf
(hopefully the entire recording will be released soon)

### Already implemented components

Note: as soon as the implementation is functional these parts will be factored out from the LES implementation and offered in a separate package or repo so that they can be easily applied for other protocols.

Client side load balancing:
https://github.com/ethereum/go-ethereum/blob/master/les/distributor.go

Token accounting:
https://github.com/ethereum/go-ethereum/blob/master/les/balance.go

Client side flow control:
https://github.com/ethereum/go-ethereum/blob/master/les/flowcontrol/control.go

Server capacity management:
https://github.com/ethereum/go-ethereum/blob/master/les/flowcontrol/manager.go
https://github.com/ethereum/go-ethereum/blob/master/les/servingqueue.go
https://github.com/ethereum/go-ethereum/blob/master/les/costtracker.go

Server balance/priority management API:
https://github.com/zsfelfoldi/go-ethereum/blob/server-api4/les/api.go

Client priority handler:
https://github.com/ethereum/go-ethereum/blob/master/les/clientpool.go

