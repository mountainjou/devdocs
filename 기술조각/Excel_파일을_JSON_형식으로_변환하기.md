# Excel 파일(.xlsx)을 JSON 형식으로 변환하기

## 프론트엔드(리액트 컴포넌트)

react-dropzon 모듈 사용
<https://github.com/react-dropzone/react-dropzone>

```js
import React, { useState, useMemo } from "react";
import { Button } from "reactstrap";
import axios from "axios";
import { useDropzone } from "react-dropzone";

// dropzone 스타일 설정
const baseStyle = {
  flex: 1,
  display: "flex",
  flexDirection: "column",
  alignItems: "center",
  padding: "20px",
  borderWidth: 2,
  borderRadius: 2,
  borderColor: "#eeeeee",
  borderStyle: "dashed",
  backgroundColor: "#fafafa",
  color: "#bdbdbd",
  outline: "none",
  transition: "border .24s ease-in-out"
};

const activeStyle = {
  borderColor: "#2196f3"
};

const acceptStyle = {
  borderColor: "#00e676"
};

const rejectStyle = {
  borderColor: "#ff1744"
};

// UploadHolders 함수형 컴포넌트 작성
const UploadHolders = props => {
  // 오피스 엑셀 파일 수락을 위한 파일 옵션
  // text/csv,application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
  const {
    acceptedFiles,
    rejectedFiles,
    getRootProps,
    getInputProps,
    isDragActive,
    isDragAccept,
    isDragReject
  } = useDropzone({
    accept:
      "text/csv,application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
  });

  const style = useMemo(
    () => ({
      ...baseStyle,
      ...(isDragActive ? activeStyle : {}),
      ...(isDragAccept ? acceptStyle : {}),
      ...(isDragReject ? rejectStyle : {})
    }),
    [isDragActive, isDragReject]
  );

  const acceptedFilesItems = acceptedFiles.map(file => (
    <li key={file.path}>
      {file.path} - {file.size} bytes
    </li>
  ));

  const rejectedFilesItems = rejectedFiles.map(file => (
    <li key={file.path}>
      {file.path} - {file.size} bytes
    </li>
  ));

  const uploadList = async () => {
    console.log(acceptedFiles[0]);
    const file = acceptedFiles[0];
    const url = "/api/upload/holderlist";
    const formData = new FormData();
    formData.append("file", file);
    const config = { headers: { "Content-Type": "multipart/form-data" } };
    const res = await axios.post(url, formData, config);
    console.log(res.data);
  };

  return (
    // <Form onSubmit={e => onSubmit(e)}>
    <section className="container">
      <div {...getRootProps({ className: "dropzone", style })}>
        <input {...getInputProps()} />
        <p>
          Drag 'n' drop stockholder's list file here, or click to select file
        </p>
      </div>
      <aside>
        <h4>Accepted file</h4>
        <ul>{acceptedFilesItems}</ul>
        <Button color="primary" onClick={uploadList}>
          Upload
        </Button>
      </aside>
    </section>
  );
};

export default UploadHolders;
```

## 백엔드(API)

작성중

```js
const express = require("express");
const router = express.Router();
const multiparty = require("multiparty");
const xlsx = require("xlsx");

require("dotenv").config();

router.post("/holderlist", async (req, res) => {
  const resData = {};

  const form = new multiparty.Form({
    autoFiles: true
  });

  form.on("file", (name, file) => {
    const workbook = xlsx.readFile(file.path);
    const sheetnames = Object.keys(workbook.Sheets);
    console.log(sheetnames);
    console.log(workbook);

    let i = sheetnames.length;

    while (i--) {
      const sheetname = sheetnames[i];
      resData[sheetname] = xlsx.utils.sheet_to_json(workbook.Sheets[sheetname]);
    }
  });

  form.on("close", () => {
    res.send(resData);
  });

  form.parse(req);
});

module.exports = router;
```

---

참조
<https://jetalog.net/58>
