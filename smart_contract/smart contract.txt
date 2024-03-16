//SPDX-License-Identifier: MIT
pragma solidity >=0.4.4 <0.7.0;
pragma experimental ABIEncoderV2;


contract voting_system { 
    
    address owner;

    uint start_voting;

    constructor() public {
        owner = msg.sender;
        start_voting = now;
    }

    mapping (string => bytes32) candidate_ID;

    mapping (string => uint) candidate_votes;
 
    string [] candidates;

    bytes32 [] voters;

     function candidateRegistration(string memory _candidateName, uint _candidateAge, string memory _candidateId) public {
        //the current time plus 1 minutes
        require(now<=(start_voting + 1 minutes), "Candidates can no longer be submitted");
        bytes32 candidate_hash = keccak256(abi.encodePacked(_candidateName, _candidateAge, _candidateId));
        candidate_ID[_candidateName] = candidate_hash;
        candidates.push(_candidateName);
        
    }

    function seeCandidates() public view returns(string[] memory) {
        return candidates;
    }

    function Vote(string memory _candidate) public {
        require(now<=start_voting + 1 minutes, "can no longer vote");
        
        bytes32 voter_hash = keccak256(abi.encodePacked(msg.sender));

        for(uint i = 0; i < voters.length; i++){
            require(voters[i] != voter_hash, "You already voted!");
        }

        voters.push(voter_hash);
        
        bool flag = false;
        
        for(uint j = 0; j < candidates.length; j++){
            if(keccak256(abi.encodePacked(candidates[j])) == keccak256(abi.encodePacked(_candidate))){
                flag=true;
            }
        }
        require(flag==true, "There is no candidate with that name");

        candidate_votes[_candidate]++;
    }
    function seeVotes(string memory _candidate) public view returns(uint) {
        return candidate_votes[_candidate];
    }

    function uint2str(uint _i) internal pure returns (string memory _uintAsString) {
        if (_i == 0) {
            return "0";
        }
        uint j = _i;
        uint len;
        while (j != 0) {
            len++;
            j /= 10;
        }
        bytes memory bstr = new bytes(len);
        uint k = len - 1;
        while (_i != 0) {
            bstr[k--] = byte(uint8(48 + _i % 10));
            _i /= 10;
        }
        return string(bstr);
    }

    function seeResults() public view returns(string memory){
        string memory results;
        
        for(uint i = 0; i < candidates.length; i++){
            results = string(abi.encodePacked(results, "(", candidates[i], ", ", uint2str(seeVotes(candidates[i])), ")---"));
        }
    
        return results;
    }

    function Winner() public view returns(string memory){

        require(now>(start_voting + 1 minutes), "Voting is not over yet.");
        
        string memory winner= candidates[0];
    
        bool flag;
        for(uint i = 1; i <candidates.length; i++){
            
            if(candidate_votes[winner] < candidate_votes[candidates[i]]){
                winner = candidates[i];
                flag=false;
            }else{
                if(candidate_votes[winner] == candidate_votes[candidates[i]]){
                    flag=true;
                }
            }
        }
        
        if(flag==true){
            winner = "There is a tie between the candidates!";
            
        }
        return winner;
    }

}

//Created
//Alhamdulillah
