pragma solidity >=0.8.4;

library time{ //own functions for working with timestamp 

    function is_leap(uint year) public returns (bool){
        if(year%4==0){
            if(year%100==0 && year%400!=0){
                return false;
            }
            return true;
        }
        return false;
    }
    
    function daysinmnt(uint month, uint year) public returns (uint){
        uint[13] memory day;
        day[1] = 31;
        day[2] = 28;
        day[3] = 31;
        day[4] = 30;
        day[5] = 31;
        day[6] = 30;
        day[7] = 31;
        day[8] = 31;
        day[9] = 30;
        day[10] = 31;
        day[11] = 30;
        day[12] = 31;
        if(is_leap(year)==true)
            day[2] = 29;
        return day[month];
    }
    
    function num_of_days(uint year, uint month, uint date) public returns (uint){
        uint sum;
        sum = (year-1970)*365 + (year-1972)/4 + 1; //ignoring that if year is divisible by 100, it has to be by 400 to be a leap year
        for(uint i = 1; i< month; i++)
        {
            sum = sum + daysinmnt(i,year);
        }
        sum = sum + date;
        return sum;
    }
    
    
    function timestamp_to_days(uint y) public returns (uint){
        return y/(24*60*60) + 1;
    } 
    
}

contract hotel_reservation{
    
    uint registrationfee;
    uint cancel_7;//percentage of fee not returned due to cancellation 7 days before booked date
    uint cancel_2;//percentage of fee not returned due to cancellation less than 2 days before booked date
    address owner;
    event registration(address guest, uint day, uint month, uint year);
    event cancellation(address guest, uint day, uint month, uint year);
    
    struct reservation{
        uint date;
        address guest;
    }
    
    mapping (uint=>address[]) private perday; //Maps all the address that have booked on a particular date to the unit containing the date 
    mapping (address=>bool) public has_cancelled;//Maps the address to a bool that stores if the guest has cancelled his/her reservation or not
    mapping (address=>uint) private balance;//Maps the address to the amount the guest has paid to us
    
    constructor()
    {
        registrationfee = 10000;
        cancel_7 = 50;
        cancel_2 = 0;
        owner = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;
    }
    
    function vacancy(uint day, uint month, uint year) public returns (bool){    
        uint _datetobook = threetoone(day,month,year);
        if (perday[_datetobook].length<100)//assuming there are 100 rooms in the hotel available per day
            return true;
        else
            return false;
    }
    
    function between(uint day, uint month, uint year) private returns (uint){ //returns number of days to resistered date for hotel
         uint current = block.timestamp;
         uint currentday = time.timestamp_to_days(current);
         uint futureday = time.num_of_days(year,month,day);
         return futureday - currentday;
    }
    
    function threetoone(uint day, uint month, uint year) private returns (uint){
        return (year*10000 + month*100 + day);
    }
    
    function remove(address unwanted, uint dater) private{
        for(uint i=0;i<perday[dater].length;i++){
            if(perday[dater][i]==unwanted){
                delete perday[dater][i];
                break;
            }
        }
        
    }
    
    function bookandpay(uint day, uint month, uint year) public payable{   //enter date you wish to book for
        uint _datetobook = threetoone(day,month,year);
        require(vacancy(day,month,year)==true,"There are no vacant rooms");
        require(msg.value==registrationfee,"You didnt pay the required amount");
        balance[msg.sender]=registrationfee;
        perday[_datetobook].push(msg.sender);
        has_cancelled[msg.sender]=false;
        emit registration(msg.sender,day,month,year);
    }
    
    function cancel(uint day, uint month, uint year) public{ //enter the date you wish to cancel the booking for
        require(has_cancelled[msg.sender]==false,"You have already cancelled you registration");
        uint daysbwt = between(day,month,year);
        if(daysbwt>7)
            payable(msg.sender).transfer(balance[msg.sender]);
        else if(daysbwt>2)
            payable(msg.sender).transfer(balance[msg.sender]*cancel_7/100);
        else if(daysbwt<=2)
            payable(msg.sender).transfer(balance[msg.sender]*cancel_2/100);
        remove(msg.sender,threetoone(day,month,year));
        balance[msg.sender] = 0;
        emit cancellation(msg.sender,day,month,year);
        has_cancelled[msg.sender]=true;
    }
    
    function change_registrationfee(uint x) public{ // allows the owner to change the registration fee
        require(msg.sender==owner,"You cannot change the registrationfee");
        registrationfee = x;
    }
    
    function change_cancel_7(uint y) public{ //allows the owner to change the cancellation fee less than a week before the registered date
        require(msg.sender==owner,"You cannot change the cancellation fee");
        cancel_7 = y;
    }
    
    function change_cancel_2(uint z) public{ // alllows the owner to change the cancellation fee less than 2 days before the registered date
        require(msg.sender==owner,"You cannot change the cancellation fee");
        cancel_2 = z;
    }

}




