

here i share some of my main focus point code that are help to solved my dynamic report generate problem.

<!--Start html page code -->
<!-- generate dynamic HTML Table with unlimited columns-->
<nb-card>
  <nb-card-body>
    <div class="table-wrapper-scroll-y my-custom-scrollbar">
      <table id="simple_table" #simple_tableData class="table table-bordered mb-0">
        <thead>
          <tr>
            <th colspan="noOfColumn" class="reportHeaderClass">{{companyAlias}} Division</th>
          </tr>
          <tr>
            <td colspan="noOfColumn" class="reportHeaderClass">{{reportTitleName}}</td>
          </tr>
          <!-- here are some parameters those are hide and show conditionally -->
          <tr *ngIf="productName">
            <td colspan="noOfColumn" class="reportHeaderClass">{{productName}}</td>
          </tr>
          <tr *ngIf="zone">
            <th colspan="noOfColumn" class="reportHeaderClass">Zone Name: {{zone}}</th>
          </tr>
          <tr *ngIf="!zone">
            <th colspan="noOfColumn" class="reportHeaderClass">Zone Name: All</th>
          </tr>
          <tr *ngIf="region">
            <th colspan="noOfColumn" class="reportHeaderClass">Region Name: {{region}}</th>
          </tr>
          <tr *ngIf="!region">
            <th colspan="noOfColumn" class="reportHeaderClass">Region Name: All</th>
          </tr>
          <tr *ngIf="areaName">
            <th colspan="noOfColumn" class="reportHeaderClass">Area Name: {{areaName}}</th>
          </tr>
          <tr *ngIf="!areaName">
            <th colspan="noOfColumn" class="reportHeaderClass">Area Name: All</th>
          </tr>
          <tr *ngIf="territory">
            <th colspan="noOfColumn" class="reportHeaderClass">Territory Name: {{territory}}</th>
          </tr>
          <tr *ngIf="!territory">
            <th colspan="noOfColumn" class="reportHeaderClass">Territory Name: All</th>
          </tr>
          <tr>
            <td colspan="noOfColumn" class="reportHeaderClass">{{dateRange}}</td>
          </tr>
          <tr>
            <td colspan="noOfColumn" class="reportHeaderClass"></td>
          </tr>
          <tr>
            <!-- generate dynamic column header name () -->
            <td *ngFor="let item of tableHeaderProp; let i = index;">{{item}}</td>
          </tr>
        </thead>
        <tbody>          
            <!-- generate dynamic column wise dynamic row value-->
          <tr *ngFor="let item of bodyData; let i = index;"
            [ngClass]="{'highlighted-row-sl-1': item.SL === 1,'highlighted-row-sl-2': item.SL ===2,'highlighted-row-sl-3': item.SL === 3,'highlighted-row-sl-4': item.SL === 4}">
            <td *ngFor="let itemx of tableHeaderProp; let i = index;">{{item[itemx]}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </nb-card-body>
</nb-card>

<!--End (url)html page code -->



<!--Start ts page code -->
//Angular Code

  @ViewChild("simple_tableData") tableSales: ElementRef;
  @ViewChild("simple_tableData", { static: false }) TABLE: ElementRef;
  @Input() subReportId: string = '';


 // call this funtion to get
  async downloadExcel() {
    //debugger
    if (this.checkReportingCriteria()) {
      await this.getReportData();
      setTimeout(() => {
        
        const workbook = new ExcelJS.Workbook();// instantiate a new excel file
        const sheet = workbook.addWorksheet("Table");// create a new sheet a excel bokk
        const table = this.tableSales.nativeElement;// get table object from html table
        const rows = table.querySelectorAll("tr");// get rows from html table

        rows.forEach((row) => {
          const excelRow = sheet.addRow([]);
          const cells = row.querySelectorAll("td");
          let rowNo = excelRow.cellCount + 1;

          cells.forEach((cell) => {
            const cellValue = cell.textContent || "";
            excelRow.getCell(rowNo + excelRow.cellCount).value = cellValue; // Set cell value
          });

          // Manually set the background color based on your CSS class
          if (row.classList.contains("highlighted-row-sl-1")) {
            excelRow.eachCell((cell) => {
              cell.fill = {
                type: "pattern",
                pattern: "solid",
                fgColor: { argb: "ffffffff" }, // Set your desired background color
              };
            });
          } else if (row.classList.contains("highlighted-row-sl-2")) {
            excelRow.eachCell((cell) => {
              cell.fill = {
                type: "pattern",
                pattern: "solid",
                fgColor: { argb: "ff3cadda" }, // Set your desired background color
              };
            });
          } else if (row.classList.contains("highlighted-row-sl-3")) {
            excelRow.eachCell((cell) => {
              cell.fill = {
                type: "pattern",
                pattern: "solid",
                fgColor: { argb: "ffa4b6a4" }, // Set your desired background color
              };
            });
          } else if (row.classList.contains("highlighted-row-sl-4")) {
            excelRow.eachCell((cell) => {
              cell.fill = {
                type: "pattern",
                pattern: "solid",
                fgColor: { argb: "b256f1a9" }, // Set your desired background color
              };
            });
          }
        });

        // set report title name 
        const reportTitleName = this.reportName
          ? `${this.reportTitleName}.xlsx`
          : "table.xlsx";

        workbook.xlsx.writeBuffer().then((buffer) => {
          const blob = new Blob([buffer], {
            type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
          });
          FileSaver.saveAs(blob, reportTitleName);
          this.isExcel = false;
        });
      }, 500);

    } else return false;
  }



// call this function to get and assign the report data sources
  getReportData(): Promise<void> {
    return new Promise((resolve, reject) => {
    
      // few variables for static report  
      this.dateRange = "";
      this.showbody = false;
      this.isPreview = true;
      this.bodyData = [];
      this.tableHeaderP = [];
      this.tableHeaderProp = [];

      this.reportName = "";
      this.reportType = "ALL";
      this.zoneCode = "";
      this.regionCode = "";
      this.areaCode = "";
      this.territoryCode = "";
      this.productWiseSpecificationId = 0;

      //conditional check for several varables value if true then assign the value.
      if (!this.isEmpty(this.reportNameSelected)) {
        this.reportName = this.reportNameSelected["id"];
      }
      if (!this.isEmpty(this.reportTypeSelected)) {
        this.reportType = this.reportTypeSelected["id"];
      }
      if (!this.isEmpty(this.reportPeriodSelected)) {
        this.reportPeriod = this.reportPeriodSelected["id"];
      }
      if (!this.isEmpty(this.zoneSelected)) {
        this.zoneCode = this.zoneSelected["id"];
      }
      if (!this.isEmpty(this.regionSelected)) {
        this.regionCode = this.regionSelected["id"];
      }
      if (!this.isEmpty(this.areaSelected)) {
        this.areaCode = this.areaSelected["id"];
      }
      if (!this.isEmpty(this.territorySelected)) {
        this.territoryCode = this.territorySelected["id"];
      }
      if (!this.isEmpty(this.masterReportSelected)) {
        this.reportMasterId = this.masterReportSelected["id"];
      }
      if (!this.isEmpty(this.productSelected)) {
        this.productWiseSpecificationId = this.productSelected["id"];
      }

      this.dateRange = "Period- " + this.cmnService.GetMonthAndYear(this.fDate) + " To " + this.cmnService.GetMonthAndYear(this.tDate);

    // call this function to get the report data sources through the API 
    this.cmnService.getPerformanceReportData().subscribe((returns: any) => {
    // response status check
        if (returns.success) {
        // data availability check
          if (returns.data.length > 0) { 
          //get the Zero index row to obtain property name for report column header
            for (var property in returns.data[0]) { 
            // avoid unnecessary property
              if (property != "SL") {  
              // get every property name for report header name and use for get cell value.
                this.tableHeaderProp.push(property); 
              }
            }
            //allow Angular to run change detection once between actions that you would otherwise perform synchronously
            setTimeout(() => {
              this.bodyData = returns.data;
              resolve();
            }, 500); //500 milliseconds
          }
        } else {
          // 
          this.toastrService.warning(this.commonService.nodatafound, "Warning");
        }
      });
    });
  }



  
<!--End ts page code -->





