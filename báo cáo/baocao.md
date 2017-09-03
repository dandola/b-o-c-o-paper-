# Báo cáo tổng hợp paper và biểu đồ khi thực hiện lookup

## paper DHT-based lightweight broadcast algorithm in large-scale computing infrastructures
### 1. giới thiệu bài báo
- Trong bài báo nói rằng có rất nhiều thuật toán sử dụng cơ chế định tuyến của DHT để đạt được khả năng mở rộng (scalability), tuy nhiên có một vấn đề vấp phải đó là mất cân bằng tải (load unbalancing) và chi phí cao trong việc xây dựng và bảo trì. bài báo này đưa ra các thuật toán DHT-based lightweight broadcast dựa trên chord để giải quyết các giới hạn trên. 

- trong bài báo này, đầu tiên là giới thiệu cơ bản về cấu trúc của Chord cũng như cách nó hoạt động.
- trong phần 2 là giới thiệu về 2 thuật DHT-based broadcast cơ bản là k-ary search based broadcast và reverse-path forwarding based broadcast
    
    + k-ary search based broadcast algorithm:với các tiếp cận theo kiểu top-down, trong thuật toán này, mỗi node chọn tất cả các node n có trong finger table với điều kiện là node n đó phải nằm trong một khoảng **limit**.
    + reverse-path forwarding based broadcast algorithm: ngược với thuật toán k-ary, với cách tiếp cận theo kiểu bottom-up, mỗi node sẽ có một bảng chưa các con của nó theo chiều hướng ngược lại.
- tuy nhiên, 2 thuật toán trên vẫn có các nhược điểm của chúng, nhược điểm chung là mất cân bằng tải, tức là có node thì có nhiều nhánh con, có nút không có nhánh nào (node lá).

- phần tiếp theo nói về 2 thuật toán của DHT-based lightweight broadcast là token-based broadcast và partition-based broadcast để giải quyết vấn đề về scalability và load balancing mà các thuật toán trước không đạt được. 2 thuật toán này đều dựa theo cách tiếp cận top-down.

    - thuật toán Token-based broadcast được sử dụng khi các node được phân tán đồng đều trong không gian định danh(identifier space), khi đó mỗi node lựa chọn các node  trong bảng finger table làm con của nó bởi một giá trị token. 
    - Thuật toán Partition-based broadcast được sử dụng khi các node được phân tán ngẫu nhiên trong không gian định danh, khi đó mỗi node sẽ chia không gian định danh của nó thành hai không gian con và lựa chọn node thích hợp trong từng không gian con làm con của nó.
- và phần cuối cùng là đưa ra thực nghiệm và kết quả để chúng minh.

### 2. Background 

### 2.1 Chord
- mạng Chord có cấu trúc bao gồm các node và các liên kết giữa các node tạo thành một vòng chord. mỗi vòng Chord thì có không gian định danh m bit. Chord sử dụng hàm băm SHA-1 để  gán các đối tượng (node hoặc data) tương tứng với một key duy nhất. Định danh của một node được băm bởi địa chỉ IP của node đó gọi là NodeID, định danh của các mục, data thì được băm bởi tên file hoặc nội dung của file... gọi là KeyID. không danh định danh được trải đều, bằng phẳng từ [ 0,2<sup>m</sup> ).
- mỗi một node có NodeID là k được quản lý bởi node có định danh id >= k, ngay sau khóa k, node này được gọi là Successor, ký hiệu succ(k). Tương tự như vậy, node ngay trước khóa k được gọi là predecessor của k, ký hiệu Pred(k). 
- Bên cạnh đó, mỗi một node u quản lý một tập các node finger, trong đó finger thứ i là finger(u,i)= succ((u + 2<sup>i</sup>)mod 2<sup>m</sup>), trong đó 0 <= i <= m-1. một tập các node finger và chỉ số như vậy được gọi là bảng **finger table**
- Chord thông qua thuật toán định tuyến để đảm bảo rằng tìm được node succ(k), node mà chứa cặp (k,v), trong đó v là giá trị của đối tượng.
- Thuật toán định tuyến của Chord cung cấp khả năng mở rộng và tìm kiếm hiệu quả, với tổng chi phí tìm kiếm là O(log n), với n là số nodes có trong chord.

### 2.2 DHT-based broadcast overview

 - Phần này trình bày 2 thuật toán cơ bản của DHT-based broacast và chỉ ra các giới hạn về khả năng mở rộng và cân bằng tải,...

 #### a. k-ary search based broadcast algorithm
 - Dựa theo tiếp cận top-down, mỗi node n lựa chọn tất cả các nodes có trong finger table trong giới hạn về không gian định danh (limit) làm con của n. Tương tự như vậy đối với node con --> sẽ tạo ra một cây DBT rooted tại một nút ban đầu.
 - Ví dụ: trong vòng chord không gian khóa là m=4, và có 16 node. với nút ban đầu là N0. limit=N0

 <img src="../báo cáo/k-ary.PNG" style="align: center" >

 - Node N0 lựa chọn các node finger là N8,N4,N2,N1 là con của N0 với khoảng giới hạn là [N0,N0], sau đó N0 gửi khoảng giới hạn là [N8,N0], [N4,N8], [N2,N4] và [N1,N2] tới lần lượt các node N8,N4,N2,N1 tương ứng, các nút con tiếp tục như trên, cho tới khi không còn nút con nào được chọn.
 - Hình dưới đây là cây DBT rooted tương ứng tại nút nguồn N0, được xây dựng bởi mô tả bên trên.

 <img src="../báo cáo/tree k-ary.PNG" >

 - từ sự mô tả và hình bên trên cho thấy thuật toán K-ary search based broadcast gây ra việc mất cân bằng tải (load unbalancing), tức là có nút có nhiều nhánh (vd: Node 8,node 4), có nút không có nhánh nào.

 #### b. reverse-path forwarding based broadcast algorithm

- Mỗi một nút đích lựa chọn đường đi tới một node nguồn theo tiếp cận từ dưới lên. khi đó cây DBT rooted tại nút nguồn được xây dựng bao gồm tất cả các đường đi định tuyến. khi broadcast một message, thuật toán gửi broadcast message dọc theo các đường định tuyến trong DBT đến tất cả các nút đích.
- Trong DBT, mỗi node cha sẽ có một bảng để lưu trữ các con theo chiều ngược như hình b dưới đây.

 <img src="../báo cáo/reverse-path.PNG" >

 - VD:xây dựng vòng chord với số nodes=16, không gian khóa m=4 bit
 - hình a thể hiện các đường đi tới node đích là N0. ví dụ: định tuyến đường đi từ N1,N5  tới N0 tương ứng là N1->N9-> N13->N15->N0 và N5->N13->N15->N0. ta thấy rằng N5,N9 cùng lựa chọn nút cha là N13, khi đó N13 cần tốn thêm chi phí cho việc lưu trữ 2 nút con là N5,N9 tương ứng
 - hình b đưa ra cây DBT rooted tại nút N0 được xây dựng thông qua hình a,bao gồm tất cả các đường đi trong hình a theo đường đi ngược lại. Mỗi nút (ngoại trừ nút lá) sẽ lưu trữ các con được mô tả bằng ô vuông như trong hình b.

 - từ sự mô tả ở trên, và hình ảnh minh họa cho thấy thuật toán reverse-path forwarding based broadcast phải chịu chi phí xây dựng và bảo trì cao, đồng thời mất cân bằng tải giữa các tải của node trong DBT.   
    
    + vì mỗi nút (trừ nút lá) có một bảng để lưu trữ các nút con, điều này gây ra tốn thêm chi phí cho việc  lưu trữ và bảo trì.
    + mỗi một nút đích khi thực hiện định tuyến đường đi, nó sử dụng thuật toán định tuyến của Chord để lựa chọn đường đi ngắn nhất, kết quả là tạo ra cây mất cân bằng trong DHT.

### 2.3 DHT-based lightweight broadcast algorithms
-Các thuật toán DHT-based lightweight broadcast được đề xuất để giải quyết vấn đề về scalability và load balancing.
- thuật toán thứ nhất là Token-Based broadcast được áp dụng trong không gian định danh là đồng đều.
- thuật toán partition-based broadcast được áp dụng trong không gian định danh có là không đồng đều (ngẫu nhiên)
#### a. Token-based broadcast algorithm

- không gian định danh đồng đều, tức là tải của các nút là giống nhau, các nút được phân bố đồng đều trong vòng Chord.
- Trong thuật toán này, mỗi node lựa chọn con trái và con phải của nó bởi một giá trị token.
- Gọi nút nguồn là Ns, destination node là Nd.

- **Thuật toán**: 
    + B1: khởi tạo một giá trị token t(t=0) tại node Ni,trong finger table của node Ni, lựa chọn  finger(Ni,t) là node con trái Nl , finger(Ni,t+1) là node con phải Nr
    + B2: Kiểm tra Nl ≠ Nr, nếu thỏa mãn thì chuyển qua bước 3. nếu không tăng t+=1, thực hiện lại bước 1.
    + B3: kiểm tra xem Dist(Nd,Nl) < Dist(Nd,Ns) và Dist(Nd,Nr) < Dist(Nd,Ns) (kiểm tra xem Nl và Nr có nằm trong không gian định danh giữa Nd và Ns hay không).  Nếu đúng, thì Nd sẽ chuyển tiếp thông điệp tới Nr và Nl với t+=1.Nếu không thì kết thúc

- dưới đây là thuật toán được minh họa bằng mã giả: 

 <img src="../báo cáo/token-base.PNG" >

 - receive(P,R,Token): chỉ ra rằng node P nhận một broadcast message với một giá trị token và nút ban đầu R.
 - Send(P,R,Token) chỉ ra một broadcast message giá trị Token và một nút nguồn ban đầu R được gửi tới node P.

 <img src="../báo cáo/chord token.PNG" >

- Hình trên đưa ra 1 ví dụ về thuật toán Token-base đối với vòng chord 16 nodes, m=4.
    + Hình a, giả sử nút ban đầu là N0 được khởi tạo với giá trị token=0 -->lựa chọn N1=finger(N0,0) và N2=finger(N0,1) là con trái và con phải vì N1 ≠ N2 -> gửi một broadcast message với token=token + 1 (t=1) tới N1,N2. tương tự như vậy cho tới khi không còn con nào được chọn, khi giá trị token=3, từ N8 tới N14 không có con nào được chọn vì định danh của các node con > node gốc N0.
    + Hình b minh họa cấu trúc cây DBT rooted được xây dựng từ N0 dựa vào hình a. với chiều cao cây là h=log<sub>2</sub>n=4

- Ưu điểm: 
    + thuật toán trên đảm bảo rằng các node khác nhau sẽ có con khác nhau, không bị chung cùng một node, do đó thuật toán token-based bao phủ toàn bộ các node đích đến, không bị trùng lặp.
    + Không mất thêm chi phí cho việc xây dựng và duy trì các node con vì nó dựa hoàn toàn vào cơ chế định tuyến và thuật toán stabilization của Chord.
    + cây DBT tạo ra đảm bảo tính scalability và load balancing.

- Nhược điểm: thuật toán được sử dụng đối với không gian khóa là đồng đều, nhưng thực tế thì không gian định danh là thay đổi, linh động. Đặc biệt, khi 1 node tham gia hoặc rời khỏi mạng, thì không gian định danh sẽ thay đổi, khi đó không gian định danh đồng đều sẽ trở thành không gian định danh không đồng đều.


#### b. partition-based broadcast algorithm
- Đối với thuật toán này, sự phân bố của các nút trong vòng có thể là ngẫu nhiên, không đồng đều, do vậy các tải giữa các nút sẽ khác nhau, điều này giải quyết được nhược điểm của thuật toán token-based broadcast.
- Trong thuật toán partition-based, mỗi node phân chia không gian khóa của nó thành hai không gian con, sau đó lựa chọn các node đại diện trong không gian con.
- Các bước để xây dựng một balanced DBT :
    + B1: khi Ni nhận một broadcast message với giá trị giới hạn là L -> Ni chia không gian khóa (Ni,L) thành 2 subspaces là (Ni,finger(Ni,j)) và [ finger(Ni,j),L), trong đó Finger(Ni,j) là node gần nhất giá trị L trong bảng finger table của Ni. 
    + B2: Ni chọn finger(Ni,k)  trong (Ni,finger(Ni,j)) là node con trái , và lựa chọn finger(Ni,j) trong (Finger(Ni,j),L) là con phải của Ni. Trong đó finger(Ni,k) là node xa nhất từ node finger(Ni,j)
    + B3: từ Ni, gửi một broadcast message với giá trị finger(Ni,j) tới nút con trái, và giá trị L tới nút con phải, rồi thực hiện lại bước 1 cho tới khi không còn node con nào được lựa chọn

- thuật toán được mô tả bằng mã giả: 
```c
    //receive(P,R,Limit) :chỉ ra rằng node P nhận một broadcast message từ nút nguồn R với giá trị giới hạn là limit.
    giá  trị khởi tạo của Limit ban đầu là R.
    
    //select right child for broadcasting
    for j=m-1 to 0 do
        if Finger[P,j] in (P,Limit) then
            Right=Finger(P,j);
            break;
        else
            Right=Null;
        endif
    endfor
    if Right=Null then
        exit(0);
    endif
    //select left child for broadcasting
    for i=0 to m-1 do
        if Finger(P,i) in (P,Right) then
            Left=Finger(P,i);
            break;
        else
            Left=Null;
        endif
    endfor
    // Eliminate child and set new limit for subspace
    if (Left ≠ Null) and (Left ≠ Right) then
        NewLimit=Right;
        send(Left,R,NewLimit);
    endif
    send(Right,R,Limit);
```
- chú ý: trong trường hợp nút left = right thì chỉ cần gửi tới nút right.
- Receive(P,R,limit) chỉ ra rằng node P nhận một broadcast message từ nút nguồn R với giá trị giới hạn là limit.
- Send(P,R,limit) : giá trị limit và nút nguồn R được gửi tới nút P 

 <img src="../báo cáo/partition.PNG" >

- hình a mô tả vòng Chord với m=4, nodes=11. nút nguồn bắt đầu là N0. giá trị limit được khởi tạo ban đầu là N0, khi đó N0 chia không gian định danh (N0,N0) thành 2 không gian con là (N0,N8) và [N8,N0), với N8 là node finger gần nhất giá trị limit L trong bảng finger table vủa N0. N0 chọn N2 thuộc (N0,N8) là con trái, và chọn N8 thuộc [N8,N0) là con phải. Sau đó gửi tới N2, N8 một broadcast message tương ứng với giá trị limit  là N8, N0.
N2,N8 thực hiện tương tự như vậy, cho tới khi không còn node con nào được chọn.
- hình b mô tả cây DBT cân bằng tương ứng tại N0 được xây dựng từ Hình a

- Ưu điểm: 
    
    + Tạo ra một cây DBT có tính Scalable và load balancing
    + không mất thêm chi phí cho việc xây dựng và bảo trì một cây DBT cân bằng
    + mỗi một nút cha không tốn chi phí cho việc lưu trữ các nút con, vì chúng không cần lưu trữ, tất cả đểu dựa vào cơ chế định tuyến và thuật toán stabilization của Chord.
    + thuật toán này tổng quát trong cả hai trường hợp là không gian khóa đồng đều và không đồng đểu.


## paper Simple Efficient Load Balancing Algorithms for Peer-To-Peer Systems

### 1. giới thiệu về bài báo
- bài báo chú trọng đưa ra 2 giao thức quan trọng về cân bằng tải 
giữa các nút có trong vòng DHT và cân bằng tải về các items được phân phối tới các nút.
- thuật toán đầu tiền là cân bằng tải, phân phối không gian khóa tới các node. điều này sẽ tạo ra một hệ thống cân bằng tải giữa các nút.
- thuật toán thứ hai mục đích là trực tiếp cân bằng các items giữa các nút khi thực hiện thêm các items vào address space, làm cho một nút bất kỳ không bị quá tải, hay chứa quá nhiều items. giúp cho items được phân bố đều trên các nút.
- nhiều hệ thống sử dụng DHT nhằm tạo ra nỗ lực để cân bằng tải:

    + mỗi item được ánh xạ ngẫu nhiên với một địa chỉ thông qua một hàm băm đủ tốt
    + mỗi node trong DHT phải chịu trách nhiệm một phần không gian địa chỉ của DHT

- Chord là một ví dụ điển hình sử dụng DHT thỏa mãn các điều kiện trên, tuy nhiên những nỗ lực tạo ra sự cân bằng tải này có thể thất bại theo 2 cách:

    + việc phân ngẫu nhiên các không gian địa chỉ (address space) tới các nodes không hoàn toàn là cân bằng. Có node được chia phần lớn, có node được chia phần bé address space 
    + có một vài ứng dụng ngăn chặn việc phân ngẫu nhiên các items tới các node. các items có thể được sắp xếp một cách rõ ràng, hoặc được gán với địa chỉ cụ thể trên ring.

- vì vậy 2 thuật toán đưa ra để giải quyết các vấn đề trên.

### 2. Address-space Balancing
- trong phần này, ta sẽ trình bày một giao thức bằng cách cải tiến consistent hashing, trong đó mỗi node chịu trách nhiệm O(1/n)  address space với xác suất cao, không sử dụng các node ảo, điều này cải thiện chi phí không gian và chi phí băng thông.
- đối với giao thức này, khi thực xóa hoặc thêm vào một node thì sẽ gây ra O(loglogn) các node khác phải thay đổi vị trí của chúng.
- Mỗi một node sẽ có một tập O(log n) vị trí tiềm năng (còn được gọi là "potenial nodes" hoặc "node ảo"). tại bất kỳ thời điểm nào, nó sẽ kích hoạt một trong số các vị trí. 
- họ chỉ rõ địa chỉ (2b+1)2<sup>-a</sup> bởi <a,b>, trong đó a,b là số nguyên thỏa mãn a >=0 and 0 <= b < 2<sup>a-1</sup>.
- họ đặt một lệnh ≺ trên các địa chỉ. nói cách khác, (a,b) ≺ (a',b') nếu a < a' hoặc (a==a' and b < b'). khi đó: 
    0= 1 ≺ 1/2 ≺ 1/4 ≺ 3/4 ≺ 1/8 ≺ 3/8 ≺ 5/8 ≺ 7/8 ≺ 1/16 ≺...

- "ideal state" (trạng thái lý tưởng): cho bất kỳ tập họp các active nodes, mỗi potential node (có thể không hoạt động) chiếm  phạm vi địa chỉ giữa nó và active node kế tiếp trên ring. mỗi node sẽ kích hoạt potenial node chiếm phạm vi địa chỉ nhỏ nhất(theo lệnh ≺).
- tại bất kỳ thời điểm nào, một node nào đó kiểm tra và quyết định một trong số O(logn) potenial nodes mà chiếm địa chỉ nhỏ nhất, node đó sẽ được kích hoạt, để cuối cùng cũng đạt trạng thái lý tưởng (ideal state).

- định lý 1: những kết luận dưới đây là đúng cho giao thức trên: 

    + có duy nhất một trạng thái lý tưởng cho bất ký tập các nút.
    + với bất kỳ trạng thái ban đầu, những cải tiến bên trong sẽ dẫn đến trạng thái lý tưởng của một mạng n nodes, 
    + trong trạng thái lý tưởng, với xác suất cao, tất cả các cặp cạnh nhau cách nhau tối đa là (4 + ε)/n, với ε <= 1/2 và c >= 1/ε<sup>2</sup>
    + với thao tác chèn hoặc xóa, sẽ có O(loglog n) nodes phải thay đổi địa chỉ để đạt tới trạng thái lý tưởng.

- trạng thái lý tưởng duy nhất được xác định như sau: trong các dãy địa chỉ được phân cách bởi dấu ≺, địa chỉ cạnh địa chỉ 1 sẽ được kích hoạt, ví nó là địa chỉ chiếm phạm vi nhỏ nhất, nên node thực của nó không có sự lựa chọn nào tốt hơn, các potenial node còn lại sẽ không được kích hoạt. trong số các potenial nodes (các node tiềm năng) còn lại, địa chỉ gần nhất địa chỉ 1/2 sẽ được lựa chọn với lý do như trên.Tiếp tục như vậy khi đó sẽ đạt được trạng thái lý tưởng.

### 3. item balancing
- phần trước, ta có đề cập tới vấn đề cân bằng tải không gian địa chỉ giữa các node, tuy nhiên điều đó vẫn chưa đủ, vì các item được phân phối một cách ngẫu nhiên, tùy ý tới không gian địa chỉ, nên có thể sẽ có các node sẽ chứa nhiều item, có node sẽ chứa ít item, có node có khi còn quá tải.
- trong phần này sẽ trình bày một giao thức để giải quyết vấn đề trên, đối với giao thức này, mỗi node được tự do di chuyển đến bất bất cứ đâu.
- gọi l<sub>i</sub> là tải của node i. trong đó i chạy từ 1,2,...n theo thứ tự các node trong không gian địa chỉ.  ε thuộc khoảng (0,1/4).
- Mỗi nút i thường liên lạc với một node j ngẫu nhiên. nếu l<sub>j</sub> <=  εl<sub>i</sub> ( giả sử l<sub>i</sub> > l<sub>j</sub> ) khi đó xảy ra 2 trường hợp:

    + TH1: i=j+1, tức là node i là successor của node j, khi đó node j tăng địa chỉ của nó cho đến khi tải của mỗi node là (l<sub>i</sub> + l<sub>j</sub>)/2
    +TH2: i ≠ j + 1 : nếu l<sub>j+1</sub> > l<sub>i</sub>, khi đó đặt i=j+1  và quay lại trường hợp 1. nếu không node j sẽ di chuyển tới giữa node i-1 và i để nắm giữ 1 nửa item của i, khi đó các item của j trước đó bây giờ sẽ được xử lý bởi node successor, tức là node j+1.

    - định lý 3: nếu mỗi node liên lạc với O(logn) ngẫu nhiên các node khác trên mỗi half-life cũng như mỗi lần tải của nó tăng gấp đổi hoặc giảm một nửa, khi đó giao thức có các thuộc tính sau:

        + với xác suất cao, tải của tất cả các node nằm giữa εL/16 và 16L/ε.
        + các item được di chuyển do sự cân bằng tải là O(1) cho mỗi item chèn hoặc xóa, và O(L) cho mỗi node chèn hoặc xóa.


## Thực nghiệm tiến hành lookup trong vòng chord

- thực nghiệm được tiến hành trên vòng Chord với số node là nodes=256, số lượng key là 4000, các node được băm bởi hàm băm SHA-1. Mỗi một node hoạt động thực hiện lookup với số lượng key là 4000. số node lỗi được tăng dần lần lượt là 40%, 50%,60%,70%,80%,90%
- hình bên trái là số lần lookup lỗi tương ứng với x% node lỗi. Từ hình vẽ cho thấy, số lượng node lỗi tăng, thì số lần lookup lỗi tăng theo
- hình bên phải là tỉ lệ lookup thành công tính theo đơn vị %. từ hình vẽ cho thấy tỉ lệ lookup thành công giảm mạnh khi khi số lượng node lỗi là 80% và 90%.

<img src="../báo cáo/lookup.PNG">