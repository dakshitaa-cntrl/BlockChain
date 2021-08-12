# BlockChain
pragma solidity >=0.8.4;
contract auction{
    
    mapping(address=>uint) private accounts;
    mapping(address=>uint) private num_of_times_called;
    mapping(address=>uint) private withdrawn_times;
    event refunded(address user);
    event bidnothighest(address bidder);
    event bidsenttobenefeciary(uint maxbid);
    address[] log;
    
    address payable benefeciary = payable(0x5B38Da6a701c568545dCfcB03FcB875f56beddC4);
    address max;
    uint end = 0;
    
    constructor(){
        accounts[max] = 0;
    }
    
    
    function bid() public payable{
        require(end==0,"The auction has ended");
        require(num_of_times_called[msg.sender]==0,"You cannot bid more than once");
        num_of_times_called[msg.sender] = 1;
        accounts[msg.sender]=msg.value;
        log.push(msg.sender);
        
    }
    
    function find_max() private{
        max = log[0];
        for(uint i = 0; i<log.length; i++){
            if(accounts[max]<accounts[log[i]]){
                max = log[i];
            }
        }
    }
    
    function withdraw() public{
        require(withdrawn_times[msg.sender]==0,"None of your money is with the contract now");
        payable(msg.sender).transfer(accounts[msg.sender]);
        withdrawn_times[msg.sender] = 1;
        accounts[msg.sender] = 0;
    }
    
    
    function check_assets() public view returns(uint){
        require(msg.sender==benefeciary,"You cannot check assets");
        return address(this).balance;
    }
    
    function end_auction() public{
        find_max();
        require(msg.sender==benefeciary,"You cannot end the auction");
        end = 1;
        payable(benefeciary).transfer(accounts[max]);
        withdrawn_times[max] = 1;
        emit bidsenttobenefeciary(accounts[max]);
    }
}


