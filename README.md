
    ret, frame = cap.read()
    if ret == False: break
    frame =  imutils.resize(frame, width=640)
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    auxFrame = frame.copy()

    faces = faceClassif.detectMultiScale(gray,1.3,5)

    for (x,y,w,h) in faces:
        cv2.rectangle(frame, (x,y),(x+w,y+h),(0,255,0),2)
        rostro = auxFrame[y:y+h,x:x+w]
        rostro = cv2.resize(rostro,(150,150),interpolation=cv2.INTER_CUBIC)
        cv2.imwrite(emotionsPath + '/rotro_{}.jpg'.format(count),rostro)
        count = count + 1
    cv2.imshow('frame',frame)

    k =  cv2.waitKey(1)
    if k == 27 or count >= 200:
        break

cap.release()
cv2.destroyAllWindows()
on:
  push:
    tags:
      - "*"

name: Deploy Extension
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - name: Publish to Open VSX Registry
        uses: HaaLeo/publish-vscode-extension@v2
        with:
          pat: ${{ secrets.OPEN_VSX_TOKEN }}
      - name: Publish to Visual Studio Marketplace
        uses: HaaLeo/publish-vscode-extension@v2
        with:
          pat: ${{ secrets.VS_MARKETPLACE_TOKEN }}
          registryUrl: https://marketplace.visualstudio.com
          - name: Publish to Open VSX Registry
  uses: HaaLeo/publish-vscode-extension@v2
  id: publishToOpenVSX
  with:
    pat: ${{ secrets.OPEN_VSX_TOKEN }}
- name: Publish to Visual Studio Marketplace
  uses: HaaLeo/publish-vscode-extension@v2
  with:
    pat: ${{ secrets.VS_MARKETPLACE_TOKEN }}
    registryUrl: https://marketplace.visualstudio.com
    extensionFile: ${{ steps.publishToOpenVSX.outputs.vsixPath }}
  
