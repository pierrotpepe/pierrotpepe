- 👋 Hi, I’m @pierrotpepe
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
pierrotpepe/pierrotpepe is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>Pierrot Pepe Token Sale</title>

    <style>
        /* Custom styles for token information section */
        .token-info {
            color: white;
            background-color: green;
            padding: 10px;
            border-radius: 5px;
            text-align: center;
        }

        /* Custom styles for buttons */
        .custom-button {
            background-color: #4CAF50; /* Green background */
            border: none;
            color: white;
            padding: 10px 20px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            font-size: 16px;
            margin: 4px 2px;
            cursor: pointer;
            border-radius: 5px;
        }

        /* Custom styles for input */
        .custom-input {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
            font-size: 16px;
            margin: 4px 2px;
            width: 200px;
            text-align: center;
        }

    </style>
</head>

<body>

    <div>
        <button id="connectButton" class="custom-button">Connect Wallet</button>
        <button id="buyButton" class="custom-button" disabled>Buy Tokens</button>
        <div class="token-info">
            <p>Current Token Price: 1 BNB = 4,206,900,000,000 PPEPE</p>
            <p>Enter Token Amount to Buy: <input type="number" id="tokenAmount" class="custom-input" min="0" oninput="displayTotalBNB()" /></p>
            <p>Total BNB Required: <span id="totalBNB"></span> BNB</p>
            <p>Your Token Balance: <span id="userBalance"></span> PPEPE</p>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/web3@1.5.2/dist/web3.min.js"></script>

    <script>
        let web3;
        let contract;
        let account;
        let isBscNetwork = false;

        async function checkNetwork() {
            if (window.ethereum) {
                try {
                    const networkId = await web3.eth.net.getId();
                    // BSC 메인넷의 네트워크 ID는 56입니다. 테스트넷인 BSC Testnet의 경우 97입니다.
                    isBscNetwork = networkId === 56 || networkId === 97;
                    return isBscNetwork;
                } catch (error) {
                    console.error("Error checking network:", error);
                    return false;
                }
            } else {
                console.error("Web3 provider not detected.");
                return false;
            }
        }

        // Function to update the connected account when it changes
        async function updateAccount() {
            const accounts = await web3.eth.getAccounts();
            account = accounts[0];
            displayUserBalance();
        }

        // Function to update the network status when it changes
        async function updateNetwork() {
            const networkChanged = await checkNetwork();
            if (!networkChanged) {
                alert("Error checking network. Please ensure you are connected to the BSC network.");
                return;
            }

            if (isBscNetwork) {
                // Enable the "Buy Tokens" button if the network is BSC
                document.getElementById("buyButton").removeAttribute("disabled");
            } else {
                // Disable the "Buy Tokens" button if the network is not BSC
                document.getElementById("buyButton").setAttribute("disabled", "true");
                alert("Please switch to the BSC network to continue with the purchase.");
            }
        }

        async function connectWallet() {
            if (window.ethereum) {
                try {
                    // 사용자의 웹3 지갑에 연결
                    await window.ethereum.request({ method: "eth_requestAccounts" });

                    // Web3.js 인스턴스 생성
                    web3 = new Web3(window.ethereum);

                    // 이벤트 리스너 등록
                    ethereum.on("accountsChanged", updateAccount);
                    ethereum.on("chainChanged", updateNetwork);

                    // 사용자의 계정 및 연결된 네트워크 업데이트
                    updateAccount();
                    updateNetwork();

                    alert("Wallet connected!");
                } catch (error) {
                    console.error("Error connecting wallet:", error);
                    alert("Error connecting wallet: " + error.message);
                }
            } else {
                alert("Web3 provider not detected. Please install a Web3-compatible wallet like MetaMask.");
            }
        }

        async function buyTokens() {
            const tokenAmount = document.getElementById("tokenAmount").value;
            const totalBNBRequired = tokenAmount / 4206900000000;
            const totalBNBRequiredInWei = web3.utils.toWei((totalBNBRequired * 10**18).toString(), "wei");
            const totalBNBRequiredInWeiBN = web3.utils.toBN(totalBNBRequiredInWei); // BigNumber로 변환

            try {
                // 가스 가격을 500분의 1로 설정 (기본값: 20 gwei)
                const gasPrice = web3.utils.toWei("3", "gwei"); // 1 gwei = 1e9 wei, 0.04 gwei = 40,000,000 wei

                // 스마트 컨트랙트 함수 호출
                const response = await contract.methods.buyTokens(totalBNBRequiredInWeiBN).send({
                    from: account,
                    value: totalBNBRequiredInWeiBN,
                    gasPrice: gasPrice, // 설정한 가스 가격을 트랜잭션에 포함
                });
                console.log("Tokens purchased:", response);
                alert("Tokens purchased successfully!");
                // 구매 후 토큰 정보를 다시 표시
                displayUserBalance();
            } catch (error) {
                console.error("Error buying tokens:", error);
                alert("Error buying tokens: " + error.message);
            }
        }

        // Connect 버튼 클릭 시 연결 함수 실행
        document.getElementById("connectButton").addEventListener("click", connectWallet);

        // 사용자의 토큰 잔액을 표시하는 함수
        async function displayUserBalance() {
            const userBalance = await contract.methods.balanceOf(account).call();
            document.getElementById("userBalance").innerText = userBalance;
        }

        // 토큰 구매에 필요한 총 BNB 금액을 표시하는 함수
        function displayTotalBNB() {
            const tokenAmount = document.getElementById("tokenAmount").value;
            const totalBNB = (tokenAmount / 4206900000000);
            document.getElementById("totalBNB").innerText = totalBNB.toFixed(5);
        }

        // Buy 버튼 클릭 시 구매 정보 표시
        document.getElementById("buyButton").addEventListener("click", function () {
            displayTotalBNB();
            buyTokens();
        });

        // 초기에 토큰 정보를 표시
        async function initialize() {
            // Web3 초기화
            if (window.ethereum) {
                try {
                    await window.ethereum.request({ method: "eth_requestAccounts" });
                    web3 = new Web3(window.ethereum);
                } catch (error) {
                    console.error("Error initializing web3:", error);
                    alert("Error initializing web3: " + error.message);
                    return;
                }
            } else {
                alert("Web3 provider not detected. Please install a Web3-compatible wallet like MetaMask.");
                return;
            }

            // 스마트 컨트랙트 연동
            const contractAddress = "0x62beB26662162B8caCF466aD3183264Bed9c2e20";
            const contractABI = [{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"owner","type":"address"},{"indexed":true,"internalType":"address","name":"spender","type":"address"},{"indexed":false,"internalType":"uint256","name":"value","type":"uint256"}],"name":"Approval","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"from","type":"address"},{"indexed":true,"internalType":"address","name":"to","type":"address"},{"indexed":false,"internalType":"uint256","name":"value","type":"uint256"}],"name":"Transfer","type":"event"},{"inputs":[{"internalType":"address","name":"owner","type":"address"},{"internalType":"address","name":"spender","type":"address"}],"name":"allowance","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"spender","type":"address"},{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"approve","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"},{"inputs":[{"internalType":"address","name":"account","type":"address"}],"name":"balanceOf","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"buyTokens","outputs":[],"stateMutability":"payable","type":"function"},{"inputs":[],"name":"decimals","outputs":[{"internalType":"uint8","name":"","type":"uint8"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"name","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"symbol","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"tokenPrice","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"totalSupply","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"recipient","type":"address"},{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"transfer","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"},{"inputs":[{"internalType":"address","name":"sender","type":"address"},{"internalType":"address","name":"recipient","type":"address"},{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"transferFrom","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"}];
            contract = new web3.eth.Contract(contractABI, contractAddress);

            // 계정 정보 가져오기
            const accounts = await web3.eth.getAccounts();
            account = accounts[0];

            // 현재 연결된 네트워크 확인
            updateNetwork();

            // Buy 버튼 활성화
            if (isBscNetwork) {
                document.getElementById("buyButton").removeAttribute("disabled");
            } else {
                document.getElementById("buyButton").setAttribute("disabled", "true");
            }

            alert("Wallet connected!");

            // 토큰 정보 표시
            displayUserBalance();
        }

        // 페이지 로드 시 initialize 함수 실행
        window.addEventListener("load", initialize);

    </script>
</body>

</html>

