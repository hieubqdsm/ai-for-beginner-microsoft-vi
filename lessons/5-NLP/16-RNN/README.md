# Mạng Nơ-ron Tái Phục Hồi

## [Câu hỏi kiểm tra trước bài giảng](https://ff-quizzes.netlify.app/en/ai/quiz/31)

Trong các phần trước, chúng ta đã sử dụng các biểu diễn ngữ nghĩa phong phú của văn bản và một bộ phân loại tuyến tính đơn giản trên các embedding. Kiến trúc này giúp nắm bắt ý nghĩa tổng hợp của các từ trong một câu, nhưng nó không xem xét đến **thứ tự** của các từ, vì phép tổng hợp trên embedding đã loại bỏ thông tin này từ văn bản gốc. Do các mô hình này không thể mô hình hóa thứ tự từ, chúng không thể giải quyết các nhiệm vụ phức tạp hoặc mơ hồ hơn như tạo văn bản hoặc trả lời câu hỏi.

Để nắm bắt ý nghĩa của chuỗi văn bản, chúng ta cần sử dụng một kiến trúc mạng nơ-ron khác, được gọi là **mạng nơ-ron tái phục hồi**, hay RNN. Trong RNN, chúng ta đưa câu qua mạng từng ký hiệu một, và mạng sẽ tạo ra một **trạng thái**, sau đó chúng ta đưa trạng thái này vào mạng cùng với ký hiệu tiếp theo.

![RNN](../../../translated_images/vi/rnn.27f5c29c53d727b5.webp)

> Hình ảnh của tác giả

Với chuỗi đầu vào các token X<sub>0</sub>,...,X<sub>n</sub>, RNN tạo ra một chuỗi các khối mạng nơ-ron và huấn luyện chuỗi này từ đầu đến cuối bằng cách sử dụng backpropagation. Mỗi khối mạng nhận một cặp (X<sub>i</sub>,S<sub>i</sub>) làm đầu vào và tạo ra S<sub>i+1</sub> làm kết quả. Trạng thái cuối cùng S<sub>n</sub> hoặc (đầu ra Y<sub>n</sub>) được đưa vào một bộ phân loại tuyến tính để tạo ra kết quả. Tất cả các khối mạng đều chia sẻ cùng trọng số và được huấn luyện từ đầu đến cuối bằng một lần backpropagation.

Vì các vector trạng thái S<sub>0</sub>,...,S<sub>n</sub> được truyền qua mạng, nó có thể học được các phụ thuộc tuần tự giữa các từ. Ví dụ, khi từ *not* xuất hiện ở đâu đó trong chuỗi, mạng có thể học cách phủ định một số phần tử trong vector trạng thái, dẫn đến phủ định.

> ✅ Vì các trọng số của tất cả các khối RNN trong hình trên được chia sẻ, hình ảnh này có thể được biểu diễn như một khối duy nhất (bên phải) với một vòng lặp phản hồi tái phục hồi, truyền trạng thái đầu ra của mạng trở lại đầu vào.

## Cấu trúc của một RNN Cell

Hãy xem cách một cell RNN đơn giản được tổ chức. Nó nhận trạng thái trước đó S<sub>i-1</sub> và ký hiệu hiện tại X<sub>i</sub> làm đầu vào, và phải tạo ra trạng thái đầu ra S<sub>i</sub> (và đôi khi, chúng ta cũng quan tâm đến một đầu ra khác Y<sub>i</sub>, như trong trường hợp các mạng tạo sinh).

Một cell RNN đơn giản có hai ma trận trọng số bên trong: một ma trận biến đổi một ký hiệu đầu vào (gọi là W), và một ma trận khác biến đổi một trạng thái đầu vào (H). Trong trường hợp này, đầu ra của mạng được tính bằng &sigma;(W&times;X<sub>i</sub>+H&times;S<sub>i-1</sub>+b), trong đó &sigma; là hàm kích hoạt và b là bias bổ sung.

<img alt="Cấu trúc RNN Cell" src="../../../translated_images/vi/rnn-anatomy.79ee3f3920b3294b.webp" width="50%"/>

> Hình ảnh của tác giả

Trong nhiều trường hợp, các token đầu vào được truyền qua lớp embedding trước khi vào RNN để giảm chiều dữ liệu. Trong trường hợp này, nếu kích thước của các vector đầu vào là *emb_size*, và vector trạng thái là *hid_size* - kích thước của W là *emb_size*&times;*hid_size*, và kích thước của H là *hid_size*&times;*hid_size*.

## Bộ Nhớ Dài Ngắn Hạn (LSTM)

Một trong những vấn đề chính của RNN cổ điển là vấn đề **gradient biến mất**. Vì RNN được huấn luyện từ đầu đến cuối trong một lần backpropagation, nó gặp khó khăn trong việc lan truyền lỗi đến các lớp đầu tiên của mạng, và do đó mạng không thể học được mối quan hệ giữa các token xa nhau. Một trong những cách để tránh vấn đề này là giới thiệu **quản lý trạng thái rõ ràng** bằng cách sử dụng các **cổng**. Có hai kiến trúc nổi tiếng thuộc loại này: **Bộ Nhớ Dài Ngắn Hạn** (LSTM) và **Đơn vị Truyền Tiếp Có Cổng** (GRU).

![Hình ảnh minh họa một cell LSTM](https://raw.githubusercontent.com/microsoft/AI-For-Beginners/main/lessons/5-NLP/16-RNN/images/long-short-term-memory-cell.svg)

> Nguồn hình ảnh TBD

Mạng LSTM được tổ chức tương tự như RNN, nhưng có hai trạng thái được truyền từ lớp này sang lớp khác: trạng thái thực tế C và vector ẩn H. Tại mỗi đơn vị, vector ẩn H<sub>i</sub> được nối với đầu vào X<sub>i</sub>, và chúng kiểm soát những gì xảy ra với trạng thái C thông qua các **cổng**. Mỗi cổng là một mạng nơ-ron với hàm kích hoạt sigmoid (đầu ra trong khoảng [0,1]), có thể được coi như một mặt nạ bitwise khi nhân với vector trạng thái. Có các cổng sau (từ trái sang phải trong hình trên):

* **Cổng quên** nhận một vector ẩn và xác định các thành phần nào của vector C cần quên và thành phần nào cần giữ lại.
* **Cổng đầu vào** lấy một số thông tin từ vector đầu vào và vector ẩn và chèn nó vào trạng thái.
* **Cổng đầu ra** biến đổi trạng thái qua một lớp tuyến tính với hàm kích hoạt *tanh*, sau đó chọn một số thành phần của nó bằng cách sử dụng vector ẩn H<sub>i</sub> để tạo ra trạng thái mới C<sub>i+1</sub>.

Các thành phần của trạng thái C có thể được coi như các cờ hiệu có thể bật và tắt. Ví dụ, khi chúng ta gặp tên *Alice* trong chuỗi, chúng ta có thể giả định rằng nó đề cập đến một nhân vật nữ và bật cờ trong trạng thái rằng chúng ta có một danh từ nữ trong câu. Khi chúng ta gặp thêm cụm từ *and Tom*, chúng ta sẽ bật cờ rằng chúng ta có một danh từ số nhiều. Do đó, bằng cách thao tác trạng thái, chúng ta có thể theo dõi các thuộc tính ngữ pháp của các phần câu.

> ✅ Một tài liệu tuyệt vời để hiểu rõ hơn về nội dung bên trong của LSTM là bài viết [Understanding LSTM Networks](https://colah.github.io/posts/2015-08-Understanding-LSTMs/) của Christopher Olah.

## RNN Hai Chiều và Nhiều Lớp

Chúng ta đã thảo luận về các mạng tái phục hồi hoạt động theo một hướng, từ đầu chuỗi đến cuối chuỗi. Điều này có vẻ tự nhiên, vì nó giống cách chúng ta đọc và nghe lời nói. Tuy nhiên, vì trong nhiều trường hợp thực tế chúng ta có thể truy cập ngẫu nhiên vào chuỗi đầu vào, có thể hợp lý khi thực hiện tính toán tái phục hồi theo cả hai hướng. Các mạng như vậy được gọi là **RNN hai chiều**. Khi làm việc với mạng hai chiều, chúng ta sẽ cần hai vector trạng thái ẩn, một cho mỗi hướng.

Một mạng tái phục hồi, dù là một chiều hay hai chiều, nắm bắt các mẫu nhất định trong một chuỗi và có thể lưu trữ chúng vào một vector trạng thái hoặc truyền vào đầu ra. Tương tự như các mạng tích chập, chúng ta có thể xây dựng một lớp tái phục hồi khác trên lớp đầu tiên để nắm bắt các mẫu cấp cao hơn và xây dựng từ các mẫu cấp thấp được trích xuất bởi lớp đầu tiên. Điều này dẫn đến khái niệm về một **RNN nhiều lớp**, bao gồm hai hoặc nhiều mạng tái phục hồi, trong đó đầu ra của lớp trước được truyền vào lớp tiếp theo làm đầu vào.

![Hình ảnh minh họa một RNN LSTM nhiều lớp](../../../translated_images/vi/multi-layer-lstm.dd975e29bb2a59fe.webp)

*Hình ảnh từ [bài viết tuyệt vời này](https://towardsdatascience.com/from-a-lstm-cell-to-a-multilayer-lstm-network-with-pytorch-2899eb5696f3) của Fernando López*

## ✍️ Bài tập: Embedding

Tiếp tục học trong các notebook sau:

* [RNNs với PyTorch](https://colab.research.google.com/github/hieubqdsm/ai-for-beginner-microsoft-vi/blob/main/lessons/5-NLP/16-RNN/RNNPyTorch.ipynb)
* [RNNs với TensorFlow](https://colab.research.google.com/github/hieubqdsm/ai-for-beginner-microsoft-vi/blob/main/lessons/5-NLP/16-RNN/RNNTF.ipynb)

## Kết luận

Trong bài học này, chúng ta đã thấy rằng RNN có thể được sử dụng cho phân loại chuỗi, nhưng thực tế, chúng có thể xử lý nhiều nhiệm vụ hơn, như tạo văn bản, dịch máy, và nhiều hơn nữa. Chúng ta sẽ xem xét các nhiệm vụ đó trong bài học tiếp theo.

## 🚀 Thử thách

Đọc qua một số tài liệu về LSTM và xem xét các ứng dụng của chúng:

- [Grid Long Short-Term Memory](https://arxiv.org/pdf/1507.01526v1.pdf)
- [Show, Attend and Tell: Neural Image Caption
Generation with Visual Attention](https://arxiv.org/pdf/1502.03044v2.pdf)

## [Câu hỏi kiểm tra sau bài giảng](https://ff-quizzes.netlify.app/en/ai/quiz/32)

## Ôn tập & Tự học

- [Understanding LSTM Networks](https://colah.github.io/posts/2015-08-Understanding-LSTMs/) của Christopher Olah.

## [Bài tập: Notebooks](assignment.md)

---

