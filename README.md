orangehrm_login_test/
    ├── pages/
    │   └── login_page.py
    ├── tests/
    │   └── test_login.py
    ├── data/
    │   └── test_data.xlsx
    ├── conftest.py
    └── requirements.txt
    selenium 
pytest 
openpyxl 
from selenium.webdriver.common.by import By 
from selenium.webdriver.support.ui import WebDriverWait 
from selenium.webdriver.support import expected_conditions as EC

class LoginPage: 
    def __init__(self, driver): 
        self.driver = driver 
        self.url = 'https://opensource-demo.orangehrmlive.com/web/index.php/auth/login'

    # Locators 
    username_locator = (By.NAME, 'username') 
    password_locator = (By.NAME, 'password') 
    login_button_locator = (By.XPATH, '//button[@type="submit"]') 
    dashboard_locator = (By.CLASS_NAME, 'oxd-topbar-header-breadcrumb')

    def open(self): 
        self.driver.get(self.url)

    def set_username(self, username): 
        element = WebDriverWait(self.driver, 10).until( 
            EC.presence_of_element_located(self.username_locator) 
        ) 
        element.clear() 
        element.send_keys(username)

    def set_password(self, password): 
        element = WebDriverWait(self.driver, 10).until( 
            EC.presence_of_element_located(self.password_locator) 
        ) 
        element.clear() 
        element.send_keys(password)

    def click_login(self): 
        element = WebDriverWait(self.driver, 10).until( 
            EC.element_to_be_clickable(self.login_button_locator) 
        ) 
        element.click()

    def verify_login_success(self): 
        return WebDriverWait(self.driver, 10).until( 
            EC.presence_of_element_located(self.dashboard_locator) 
        ) is not None 
    import pytest 
from selenium import webdriver 
from openpyxl import load_workbook 
from pages.login_page import LoginPage

@pytest.fixture(scope='module') 
def driver(): 
    driver = webdriver.Chrome(executable_path='/path/to/chromedriver')  # Update the path 
    yield driver 
    driver.quit()

def get_test_data(file_path): 
    workbook = load_workbook(file_path) 
    sheet = workbook.active 
    data = [] 
    for row in range(2, sheet.max_row + 1): 
        data.append({ 
            'ID': sheet.cell(row, 1).value, 
            'Username': sheet.cell(row, 2).value, 
            'Password': sheet.cell(row, 3).value, 
            'Date': sheet.cell(row, 4).value, 
            'Time': sheet.cell(row, 5).value, 
            'Tester': sheet.cell(row, 6).value, 
            'TestResult': sheet.cell(row, 7).value, 
        }) 
    return data

def write_test_result(file_path, row_num, result): 
    workbook = load_workbook(file_path) 
    sheet = workbook.active 
    sheet.cell(row_num, 7, result) 
    workbook.save(file_path)

@pytest.mark.parametrize( 
    "test_data", 
    get_test_data('data/test_data.xlsx') 
) 
def test_login(driver, test_data): 
    login_page = LoginPage(driver) 
    login_page.open() 
    login_page.set_username(test_data['Username']) 
    login_page.set_password(test_data['Password']) 
    login_page.click_login()

    passed = login_page.verify_login_success() 
    write_test_result('data/test_data.xlsx', test_data['ID'] + 1, 'Passed' if passed else 'Failed') 
    assert passed 
    import pytest 
from selenium import webdriver

@pytest.fixture(scope="session") 
def browser(): 
    driver = webdriver.Chrome(executable_path='/path/to/chromedriver')  # Update the path 
    yield driver 
    driver.quit() 
