# BÀI 1: PHÂN TÍCH & LỰA CHỌN PROMPT VIẾT HÀM/CODE SNIPPET

## 1. Đáp án lựa chọn

**Chọn phương án B.**

Phương án B là prompt tối ưu nhất vì mô tả rõ vai trò của AI, phiên bản ngôn ngữ Java cần dùng, đầu vào, đầu ra, cấu trúc dữ liệu, logic xử lý, ràng buộc kiểm duyệt dữ liệu và yêu cầu xử lý lỗi XML. Ngoài ra, phương án này còn yêu cầu AI thực hiện **pseudocode** và **dry-run** trước khi sinh mã nguồn, giúp giảm nguy cơ AI viết code sai logic hoặc bỏ sót trường hợp biên.

---

## 2. Phân tích vì sao phương án B tối ưu nhất

Phương án B tối ưu vì có đầy đủ 4 cấu phần kỹ thuật quan trọng khi viết prompt yêu cầu AI sinh code: **Input, Processing, Output, Language**.

### 2.1. Input – Đầu vào rõ ràng

Prompt B mô tả cụ thể đầu vào là:

> Một chuỗi XML đại diện cho danh sách giao dịch.

Mỗi giao dịch gồm các thẻ:

* `<id>`
* `<amount>`
* `<status>`

Việc mô tả rõ cấu trúc XML giúp AI hiểu chính xác dữ liệu cần đọc, tránh viết hàm chung chung hoặc tự suy đoán sai định dạng.

---

### 2.2. Processing – Logic xử lý cụ thể

Prompt B nêu rõ các bước xử lý cần có:

* Sử dụng thư viện XML tích hợp sẵn của Java như DOM hoặc SAX Parser.
* Đọc từng nút giao dịch trong XML.
* Ánh xạ dữ liệu sang `TransactionDTO`.
* Kiểm tra điều kiện biên:

   * `amount` phải lớn hơn 0.
   * `id` không được null hoặc rỗng.
* Nếu dữ liệu không hợp lệ thì ném `InvalidTransactionException`.
* Nếu XML bị lỗi định dạng thì phải bắt và xử lý ngoại lệ an toàn.

Đây là phần rất quan trọng vì yêu cầu của bài không chỉ là “đọc XML”, mà còn phải kiểm soát lỗi phân tích cú pháp và dữ liệu biên.

---

### 2.3. Output – Đầu ra xác định rõ

Prompt B yêu cầu đầu ra là:

```java
List<TransactionDTO>
```

Trong đó, `TransactionDTO` được định nghĩa là Java record:

```java
record TransactionDTO(String id, double amount, String status)
```

Nhờ vậy, AI không tự tạo sai kiểu dữ liệu, không trả về `Map`, `String`, `Object` hoặc ghi thẳng vào database.

---

### 2.4. Language – Ngôn ngữ và môi trường rõ ràng

Prompt B yêu cầu rõ:

* Sử dụng **Java 17**.
* Dùng **Java record**.
* Dùng thư viện XML tích hợp sẵn của Java như DOM hoặc SAX Parser.

Điều này giúp code sinh ra phù hợp với môi trường thực tế, không phụ thuộc thư viện ngoài không cần thiết.

---

## 3. Vai trò của kỹ thuật Dry-run CoT trong prompt B

Phương án B yêu cầu AI không viết code ngay mà phải làm theo quy trình:

1. Phác thảo thuật toán bằng pseudocode.
2. Dry-run với ca dữ liệu lỗi, ví dụ XML thiếu thẻ đóng `</id>`.
3. Cuối cùng mới sinh mã Java hoàn chỉnh.

Cách này giúp kiểm tra trước tư duy xử lý lỗi của AI. Ví dụ, nếu XML bị thiếu thẻ đóng `</id>`, bộ phân tích cú pháp XML sẽ phát hiện lỗi cấu trúc và ném ra ngoại lệ như `SAXException`. Khi đó chương trình cần bắt lỗi này và chuyển thành `InvalidTransactionException`.

Nhờ dry-run, người dùng có thể kiểm tra xem AI có thực sự hiểu yêu cầu hay chỉ viết code theo mẫu.

---

## 4. Lý do loại trừ các phương án còn lại

### 4.1. Loại phương án A

Phương án A:

> "Viết hộ tôi một hàm Java đọc chuỗi XML giao dịch và trả về danh sách TransactionDTO. Hãy bắt các lỗi dữ liệu."

Phương án này quá chung chung và thiếu nhiều thông tin quan trọng.

Nhược điểm:

* Không nêu rõ phiên bản Java.
* Không mô tả cấu trúc XML gồm những thẻ nào.
* Không định nghĩa rõ `TransactionDTO`.
* Không nêu cụ thể điều kiện hợp lệ của `amount` và `id`.
* Không yêu cầu xử lý lỗi XML Parsing.
* Không yêu cầu dry-run để kiểm tra logic trước khi viết code.

Nếu dùng prompt A, AI có thể viết code thiếu kiểm tra dữ liệu biên, không xử lý XML sai định dạng hoặc tự suy đoán sai cấu trúc đầu vào.

---

### 4.2. Loại phương án C

Phương án C:

> "Hãy viết một class Java sử dụng Java Stream API song song và Spring XML Reader để đọc chuỗi XML giao dịch với hiệu năng cực cao, bỏ qua các dòng lỗi và lưu thẳng vào database."

Phương án này không phù hợp với yêu cầu đề bài.

Nhược điểm:

* Tự ý thêm Spring XML Reader, trong khi đề bài chỉ yêu cầu Java 17 và có thể dùng thư viện XML tích hợp sẵn.
* Tự ý yêu cầu hiệu năng cực cao, dễ làm AI tập trung sai trọng tâm.
* Yêu cầu bỏ qua các dòng lỗi, trong khi đề bài yêu cầu giao dịch không hợp lệ phải ném `InvalidTransactionException`.
* Tự ý lưu thẳng vào database, trong khi đề bài chỉ yêu cầu map thành danh sách `TransactionDTO`.
* Dùng Stream API song song không cần thiết, thậm chí có thể gây phức tạp khi xử lý XML và lỗi dữ liệu.

Vì vậy, phương án C có nguy cơ làm AI viết sai hoàn toàn yêu cầu nghiệp vụ.

---

## 5. Mã nguồn Java hoàn chỉnh dựa trên prompt tối ưu

```java
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;

import javax.xml.XMLConstants;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import java.io.IOException;
import java.io.StringReader;
import java.util.ArrayList;
import java.util.List;

public class TransactionXmlMapper {

    public static List<TransactionDTO> parseTransactions(String xml) throws InvalidTransactionException {
        if (xml == null || xml.isBlank()) {
            throw new InvalidTransactionException("Input XML must not be null or blank.");
        }

        try {
            DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();

            // Tăng an toàn khi parse XML, hạn chế XXE attack
            factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
            factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
            factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
            factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
            factory.setExpandEntityReferences(false);

            DocumentBuilder builder = factory.newDocumentBuilder();

            Document document = builder.parse(new InputSource(new StringReader(xml)));
            document.getDocumentElement().normalize();

            NodeList transactionNodes = document.getElementsByTagName("transaction");
            List<TransactionDTO> transactions = new ArrayList<>();

            for (int i = 0; i < transactionNodes.getLength(); i++) {
                Node node = transactionNodes.item(i);

                if (node.getNodeType() == Node.ELEMENT_NODE) {
                    Element transactionElement = (Element) node;

                    String id = getRequiredTagValue(transactionElement, "id");
                    String amountText = getRequiredTagValue(transactionElement, "amount");
                    String status = getRequiredTagValue(transactionElement, "status");

                    if (id == null || id.isBlank()) {
                        throw new InvalidTransactionException("Transaction id must not be null or blank.");
                    }

                    double amount;
                    try {
                        amount = Double.parseDouble(amountText);
                    } catch (NumberFormatException e) {
                        throw new InvalidTransactionException("Transaction amount must be a valid number: " + amountText, e);
                    }

                    if (amount <= 0) {
                        throw new InvalidTransactionException("Transaction amount must be greater than 0. Invalid amount: " + amount);
                    }

                    transactions.add(new TransactionDTO(id.trim(), amount, status.trim()));
                }
            }

            return transactions;

        } catch (ParserConfigurationException e) {
            throw new InvalidTransactionException("XML parser configuration error.", e);
        } catch (SAXException e) {
            throw new InvalidTransactionException("Invalid XML format.", e);
        } catch (IOException e) {
            throw new InvalidTransactionException("Error while reading XML input.", e);
        }
    }

    private static String getRequiredTagValue(Element parent, String tagName) throws InvalidTransactionException {
        NodeList nodeList = parent.getElementsByTagName(tagName);

        if (nodeList == null || nodeList.getLength() == 0) {
            throw new InvalidTransactionException("Missing required tag: " + tagName);
        }

        Node node = nodeList.item(0);

        if (node == null || node.getTextContent() == null) {
            throw new InvalidTransactionException("Empty required tag: " + tagName);
        }

        return node.getTextContent().trim();
    }
}

record TransactionDTO(String id, double amount, String status) {
}

class InvalidTransactionException extends Exception {
    public InvalidTransactionException(String message) {
        super(message);
    }

    public InvalidTransactionException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

---

## 6. Kết luận

Tóm lại, phương án B là lựa chọn tối ưu nhất vì prompt có cấu trúc rõ ràng, đầy đủ đầu vào, đầu ra, logic xử lý, ngôn ngữ lập trình và yêu cầu kiểm thử tư duy bằng dry-run. So với phương án A quá mơ hồ và phương án C đi sai yêu cầu, phương án B giúp AI sinh ra mã nguồn chính xác, an toàn và phù hợp nhất với bài toán đồng bộ giao dịch trong hệ thống QuickPay.
