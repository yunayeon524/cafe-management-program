# cafe-management-program
카페 주문 및 매장 관리 프로그램
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <locale.h>
#include <time.h>

#define MAX_PRODUCTS 100
#define MAX_CART 100
#define MAX_ORDERS 1000
#define NAME_LEN 60
#define DATA_FILE "products.dat"
#define ORDER_FILE "orders.dat"

typedef struct {
    int id;
    char name[NAME_LEN];
    int price;
    int stock;
    int active;
} Product;

typedef struct {
    int productId;
    char productName[NAME_LEN];
    int quantity;
    int unitPrice;
    int subtotal;
} OrderItem;

typedef struct {
    int orderNo;
    char dateTime[32];
    int itemCount;
    OrderItem items[MAX_CART];
    int total;
} Order;

Product products[MAX_PRODUCTS];
Order orders[MAX_ORDERS];
int productCount = 0;
int orderCount = 0;
int nextProductId = 1;
int nextOrderNo = 1;

/* ---------- 공통 입력 함수 ---------- */

void readLine(const char *prompt, char *buffer, size_t size) {
    int ch;
    printf("%s", prompt);
    if (fgets(buffer, (int)size, stdin) == NULL) {
        buffer[0] = '\0';
        return;
    }

    size_t len = strlen(buffer);
    if (len > 0 && buffer[len - 1] == '\n') {
        buffer[len - 1] = '\0';
    } else {
        while ((ch = getchar()) != '\n' && ch != EOF) { }
    }
}

int readInt(const char *prompt, int minValue, int maxValue) {
    char buffer[128];
    char *end;
    long value;

    while (1) {
        readLine(prompt, buffer, sizeof(buffer));
        value = strtol(buffer, &end, 10);

        while (*end == ' ' || *end == '\t') end++;

        if (buffer[0] != '\0' && *end == '\0' &&
            value >= minValue && value <= maxValue) {
            return (int)value;
        }
        printf("%d부터 %d 사이의 정수를 입력해 주세요.\n", minValue, maxValue);
    }
}

void pauseScreen(void) {
    char temp[8];
    readLine("\n계속하려면 Enter를 누르세요...", temp, sizeof(temp));
}

int findProductIndexById(int id) {
    for (int i = 0; i < productCount; i++) {
        if (products[i].id == id && products[i].active) return i;
    }
    return -1;
}

/* ---------- 파일 저장 및 불러오기 ---------- */

void saveProducts(void) {
    FILE *fp = fopen(DATA_FILE, "wb");
    if (fp == NULL) {
        printf("상품 정보를 저장하지 못했습니다.\n");
        return;
    }
    fwrite(&productCount, sizeof(int), 1, fp);
    fwrite(&nextProductId, sizeof(int), 1, fp);
    fwrite(products, sizeof(Product), productCount, fp);
    fclose(fp);
}

void saveOrders(void) {
    FILE *fp = fopen(ORDER_FILE, "wb");
    if (fp == NULL) {
        printf("주문 내역을 저장하지 못했습니다.\n");
        return;
    }
    fwrite(&orderCount, sizeof(int), 1, fp);
    fwrite(&nextOrderNo, sizeof(int), 1, fp);
    fwrite(orders, sizeof(Order), orderCount, fp);
    fclose(fp);
}

void loadProducts(void) {
    FILE *fp = fopen(DATA_FILE, "rb");
    if (fp == NULL) {
        /* 최초 실행 시 기본 메뉴 등록 */
        Product defaults[] = {
            {1, "아메리카노", 3000, 30, 1},
            {2, "카페라떼", 4000, 25, 1},
            {3, "바닐라라떼", 4500, 20, 1},
            {4, "초코라떼", 4500, 15, 1},
            {5, "레몬에이드", 4300, 18, 1}
        };
        productCount = (int)(sizeof(defaults) / sizeof(defaults[0]));
        memcpy(products, defaults, sizeof(defaults));
        nextProductId = 6;
        saveProducts();
        return;
    }

    int count = 0;
    int nextId = 1;
    if (fread(&count, sizeof(int), 1, fp) != 1 ||
        int count = 0;
    int nextNo = 1;
    if (fread(&count, sizeof(int), 1, fp) != 1 ||
        fread(&nextNo, sizeof(int), 1, fp) != 1 ||
        count < 0 || count > MAX_ORDERS ||
        fread(orders, sizeof(Order), count, fp) != (size_t)count) {
        printf("주문 내역 파일을 읽을 수 없어 새 기록으로 시작합니다.\n");
        orderCount = 0;
        fclose(fp);
        return;
    }

    orderCount = count;
    nextOrderNo = nextNo;
    fclose(fp);
}

/* ---------- 메뉴 출력 ---------- */

void printMenu(void) {
    printf("\n================ 카페 메뉴 ================\n");
    printf("%-5s %-20s %-10s %-10s\n", "번호", "메뉴", "가격(원)", "재고");
    printf("--------------------------------------------\n");

    int found = 0;
    for (int i = 0; i < productCount; i++) {
        if (!products[i].active) continue;
        found = 1;
        printf("%-5d %-20s %-10d ", products[i].id,
               products[i].name, products[i].price);
        if (products[i].stock <= 0) printf("%-10s\n", "품절");
        else printf("%-10d\n", products[i].stock);
    }
    if (!found) printf("등록된 메뉴가 없습니다.\n");
    printf("============================================\n");
}

/* ---------- 고객 주문 기능 ---------- */

void customerOrder(void) {
    Order current;
    memset(&current, 0, sizeof(current));
    printf("\n[고객 주문]\n");

    while (1) {
        printMenu();
        printf("장바구니 담기를 마치려면 0을 입력하세요.\n");
        int id = readInt("주문할 메뉴 번호: ", 0, 1000000);
        if (id == 0) break;

        int index = findProductIndexById(id);
        if (index == -1) {
            printf("존재하지 않는 메뉴 번호입니다.\n");
            continue;
        }
        if (products[index].stock <= 0) {
            printf("해당 메뉴는 품절입니다.\n");
            continue;
        }

        int quantity = readInt("수량: ", 1, 10000);
                    if (current.items[i].productId == id) {
                cartIndex = i;
                break;
            }
        }

        int alreadyAdded = (cartIndex >= 0) ? current.items[cartIndex].quantity : 0;
        if (quantity + alreadyAdded > products[index].stock) {
            printf("재고가 부족합니다. 현재 재고: %d개\n", products[index].stock);
            continue;
        }

        if (cartIndex == -1) {
            if (current.itemCount >= MAX_CART) {
                printf("장바구니에 더 이상 담을 수 없습니다.\n");
                continue;
            }
            cartIndex = current.itemCount++;
            current.items[cartIndex].productId = id;
            snprintf(current.items[cartIndex].productName, NAME_LEN, "%s",
                     products[index].name);
            current.items[cartIndex].unitPrice = products[index].price;
            current.items[cartIndex].quantity = 0;
        }

        current.items[cartIndex].quantity += quantity;
        current.items[cartIndex].subtotal =
            current.items[cartIndex].unitPrice * current.items[cartIndex].quantity;

        printf("%s %d개를 장바구니에 담았습니다.\n",
               products[index].name, quantity);
    }

    if (current.itemCount == 0) {
        printf("주문이 취소되었습니다.\n");
        return;
    }

    current.total = 0;
    printf("\n================ 주문 내역 ================\n");
    for (int i = 0; i < current.itemCount; i++) {
        printf("%-18s %3d개 x %6d원 = %7d원\n",
               current.items[i].productName,
               current.items[i].quantity,
               current.items[i].unitPrice,
               current.items[i].subtotal);
        current.total += current.items[i].subtotal;
    }
    printf("--------------------------------------------\n");
    printf("총 결제 금액: %d원\n", current.total);

    int confirm = readInt("결제하시겠습니까? (1: 결제 / 0: 취소): ", 0, 1);
    if (confirm == 0) {
        printf("주문이 취소되었습니다. 재고는 차감되지 않았습니다.\n");
        return;
    }

    if (orderCount >= MAX_ORDERS) {
        printf("주문 기록 공간이 가득 차 주문할 수 없습니다.\n");
        return;
    }

    /* 결제 직전 재고를 다시 확인한 뒤 한 번에 차감 */
    for (int i = 0; i < current.itemCount; i++) {
        int index = findProductIndexById(current.items[i].productId);
        if (index == -1 || products[index].stock < current.items[i].quantity) {
            printf("재고가 변경되어 주문을 완료할 수 없습니다.\n");
            return;
        }
    }

    for (int i = 0; i < current.itemCount; i++) {
        int index = findProductIndexById(current.items[i].productId);
        products[index].stock -= current.items[i].quantity;
    }

    current.orderNo = nextOrderNo++;
    time_t now = time(NULL);
    struct tm *timeInfo = localtime(&now);
    if (timeInfo != NULL) {
        strftime(current.dateTime, sizeof(current.dateTime),
                 "%Y-%m-%d %H:%M:%S", timeInfo);
    } else {
        snprintf(current.dateTime, sizeof(current.dateTime), "날짜 정보 없음");
    }

    orders[orderCount++] = current;
    saveProducts();
    saveOrders();

    printf("\n결제가 완료되었습니다!\n");
    printf("주문 번호: %d\n", current.orderNo);
    printf("남은 재고가 자동으로 반영되었습니다.\n");
}

/* ---------- 관리자 기능 ---------- */

void addProduct(void) {
    if (productCount >= MAX_PRODUCTS) {
        printf("더 이상 메뉴를 등록할 수 없습니다.\n");
        return;
    }

    Product p;
    memset(&p, 0, sizeof(p));
    p.id = nextProductId++;
    p.active = 1;

    readLine("새 메뉴 이름: ", p.name, sizeof(p.name));
            /* 기존 주문 기록과의 연결을 위해 실제 배열에서 제거하지 않고 비활성화 */
        products[index].active = 0;
        saveProducts();
        printf("메뉴가 삭제 처리되었습니다.\n");
    } else {
        printf("삭제가 취소되었습니다.\n");
    }
}

void adjustStock(void) {
    printMenu();
    int id = readInt("재고를 조정할 메뉴 번호: ", 1, 1000000);
    int index = findProductIndexById(id);
    if (index == -1) {
        printf("해당 메뉴를 찾을 수 없습니다.\n");
        return;
    }

    printf("현재 재고: %d개\n", products[index].stock);
    int newStock = readInt("변경할 재고 수량: ", 0, 1000000);
    products[index].stock = newStock;
    saveProducts();
    printf("재고가 변경되었습니다.\n");
}

void showSalesAndOrders(void) {
    printf("\n================ 주문 및 매출 내역 ================\n");
    if (orderCount == 0) {
        printf("저장된 주문 내역이 없습니다.\n");
        return;
    }

    long long totalSales = 0;
    for (int i = 0; i < orderCount; i++) {
        totalSales += orders[i].total;
        printf("\n[주문 번호 %d] %s\n", orders[i].orderNo, orders[i].dateTime);
        for (int j = 0; j < orders[i].itemCount; j++) {
            printf("  %s / %d개 / %d원\n",
                   orders[i].items[j].productName,
                   orders[i].items[j].quantity,
                   orders[i].items[j].subtotal);
        }
        printf("  주문 금액: %d원\n", orders[i].total);
    }
    printf("\n누적 매출: %lld원\n", totalSales);
    printf("총 주문 건수: %d건\n", orderCount);
    printf("====================================================\n");
}

void adminMenu(void) {
    while (1) {
        printf("\n============== 관리자 시스템 ==============\n");
        printf("1. 메뉴 목록 및 재고 확인\n");
        printf("2. 메뉴 등록\n");
        printf("3. 메뉴 수정\n");
        printf("4. 메뉴 삭제\n");
        printf("5. 재고 수량 조정\n");
        printf("6. 주문 내역 및 누적 매출 확인\n");
        printf("0. 이전 메뉴로\n");
        printf("============================================\n");

        int choice = readInt("선택: ", 0, 6);
        switch (choice) {
            case 1: printMenu(); break;
            case 2: addProduct(); break;
            case 3: editProduct(); break;
            case 4: deleteProduct(); break;
            case 5: adjustStock(); break;
            case 6: showSalesAndOrders(); break;
            case 0: return;
        }
        pauseScreen();
    }
}

/* ---------- 메인 프로그램 ---------- */

int main(void) {
    setlocale(LC_ALL, "");

#ifdef _WIN32
    /* Windows 터미널에서 UTF-8 한글 출력 시도 */
    system("chcp 65001 > nul");
#endif

    loadProducts();
    loadOrders();

    printf("============================================\n");
    printf("          CAFE ORDER & STORE MANAGER        \n");
    printf("             카페 주문 및 매장 관리         \n");
    printf("============================================\n");

    while (1) {
        printf("\n=============== 메인 메뉴 ===============\n");
        printf("1. 고객 주문 시스템\n");
        printf("2. 관리자 시스템\n");
        printf("0. 프로그램 종료\n");
        printf("==========================================\n");

        int choice = readInt("메뉴 선택: ", 0, 2);
        switch (choice) {
            case 1:
                customerOrder();
                pauseScreen();
                break;
            case 2:
                adminMenu();
                break;
            case 0:
                saveProducts();
                saveOrders();
                printf("프로그램을 종료합니다. 이용해 주셔서 감사합니다.\n");
                return 0;
        }
    }
}
