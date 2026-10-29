# Audio Engine — nhạc, hiệu ứng âm thanh và âm nền

Audio Engine khiến game có hơi thở âm thanh: nhạc đi theo cảnh hiện tại, hiệu ứng phản hồi hành động, còn một vòng lặp âm nền đặt tông cho địa điểm. Các lớp này có thanh âm lượng riêng, nên bạn có thể hạ nhạc nền mà vẫn nghe rõ tiếng bấm giao diện và âm thanh môi trường.

Đây là mod engine dùng chung cho menu và các game. Thông thường bạn không mở hay vận hành Audio Engine trực tiếp: game cung cấp tệp âm thanh cho nó và nhờ nó phát.

![Ba kênh phát: playlist của cảnh nạp cho Nhạc, hiệu ứng hành động nạp cho Hiệu ứng, vòng lặp âm nền của cảnh nạp cho Âm nền. Mỗi kênh có âm lượng và cờ bật riêng.](assets/audio-channels.png)

## Bạn nghe được gì

- **Nhạc:** playlist của cảnh cung cấp các bài hát. Khi một bài kết thúc, trình phát chọn bài khác và tránh lặp lại bài vừa phát nếu còn lựa chọn khác. Danh sách chỉ có một bài thì lặp lại bài đó.
- **Hiệu ứng:** những âm ngắn như tiếng bấm giao diện hay hành động chiến đấu. Hiệu ứng thuộc các nhãn âm khác nhau có thể chồng lên nhau; hiệu ứng mới cùng nhãn sẽ thay thế hiệu ứng trước.
- **Âm nền:** âm thanh nền của một địa điểm, ví dụ không khí quán rượu từ Town Pack. Cùng một lúc chỉ phát một luồng âm nền và lặp lại cho tới khi bị dừng hay thay thế.

Nhạc và âm nền là hai thứ khác nhau dù cả hai đều phát nền. Một địa điểm có thể có nhạc khám phá trên kênh Nhạc cùng lúc với một vòng lặp phòng hay ngoài trời riêng trên Âm nền.

## Chỉnh âm thanh

Mở **Cài đặt → Âm thanh** trong menu. Ba hàng là **Hiệu ứng âm thanh**, **Nhạc** và **Âm nền**. Mỗi hàng có cờ bật và thanh trượt âm lượng riêng. Thay đổi được lưu ngay khi bạn thực hiện.

![Thẻ âm thanh trong Cài đặt: các hàng Hiệu ứng âm thanh, Nhạc và Âm nền, mỗi hàng có cờ bật/tắt và thanh trượt âm lượng. Ở đây Nhạc bị hạ xuống 1% trong khi Hiệu ứng và Âm nền giữ 100%.](assets/sound-settings.png)

Ví dụ, hạ **Nhạc** xuống để nghe rõ hiệu ứng gắn với hội thoại hay giao diện mà không đụng vào **Hiệu ứng âm thanh**. Tắt **Âm nền** nếu chỉ muốn nhạc không kèm vòng lặp môi trường.

Bảng cài đặt do mod Menu cung cấp; Audio Engine áp các tùy chọn đó vào việc phát. Nó không thêm một cửa sổ cài đặt âm thanh nào khác.

## Được dùng ở đâu

- Frontend dùng chung nạp Audio Engine cho menu chính và cài đặt, kể cả khi không có game nào đang chạy.
- Music Pack cung cấp chủ đề menu và các playlist khám phá/chiến đấu dùng chung; Audio Engine phát chúng.
- Trong mẫu tiểu thuyết, cảnh quán rượu yêu cầu `bar/bar_ambient`, do Town Pack cung cấp.
- Hệ thống cốt truyện và chiến đấu theo tiến trình game mà yêu cầu hiệu ứng hay đổi ngữ cảnh nhạc đang dùng.

**Audio Engine và Music Pack có mục đích khác nhau.** Audio Engine lo phần phát; Music Pack lo một soundtrack cụ thể và các playlist. Tác giả hoàn toàn có thể dùng Audio Engine với nội dung âm thanh khác.

## Cho tác giả game: thêm soundtrack

Thêm `audio_base>=1` vào danh sách `depends` sẵn có trong manifest của mod nội dung. Đồng thời cài mod và đưa nó vào bộ sưu tập của game định nhắm tới: mod nội dung đã cài không tự động thuộc về mọi game.

```jsonc
// Thêm vào danh sách phụ thuộc sẵn có của manifest:
"depends": ["audio_base>=1"]
```

Đặt nhạc dưới `music/`, còn hiệu ứng hay vòng lặp âm nền dưới `sounds/`. Với ví dụ mới hãy dùng Ogg Vorbis (`.ogg`); soundtrack và âm quán rượu kèm sẵn đều ở định dạng này.

```filestree
my_audio_content/
├── manifest.json
├── music/
│   ├── evening_theme.ogg
│   └── playlists/
│       └── my_scene.json
└── sounds/
    └── my_room/
        └── ambience.ogg
```

Một ví dụ `music/playlists/my_scene.json`:

```json
{
  "Scene": "my_scene",
  "Musics": [
    {
      "MusicType": "Ambient",
      "MusicNames": ["evening_theme"]
    }
  ]
}
```

`Scene` phải khớp với tên logic của cảnh. Tên bài hát tính tương đối từ `music/`, không kèm phần mở rộng. Giữ nguyên chính tả, chữ hoa chữ thường và khoảng cách đúng như tên tài nguyên.

Ở đây `MusicType: "Ambient"` nghĩa là **nhạc** khám phá. Nó không đẩy bài hát sang kênh Âm nền riêng biệt.

## Cho tác giả game: cho cảnh một vòng lặp âm nền

Đặt `defaultAmbient` trong định nghĩa cảnh thành một tài nguyên âm thanh tính tương đối từ `sounds/`, không kèm phần mở rộng:

```json
{
  "name": "my_scene",
  "defaultAmbient": "my_room/ambience"
}
```

Đoạn này minh họa các trường liên quan tới âm thanh; hãy giữ lại những trường khác mà cảnh của bạn cần. Một cảnh có `defaultAmbient` rỗng sẽ dừng vòng lặp âm nền của cảnh trước.

Một ví dụ thật là cảnh `my_novel_bar` trong mẫu tiểu thuyết: nó dùng `defaultAmbient: "bar/bar_ambient"` với âm thanh do Town Pack cung cấp. Dùng lại tài nguyên đó trong mod của bạn cũng đòi hỏi phụ thuộc vào chủ nhân của nó, `town_pack`.

## Đổi cảnh, chiến đấu và ghi đè

- Nếu cảnh không có playlist dùng được, Audio Engine sẽ tìm playlist dùng chung `default`. Điều này chỉ sinh ra nhạc khi có nội dung âm thanh tương ứng.
- Khi cảnh đã có playlist riêng, nó không được lấp đầy các loại nhạc còn thiếu từ `default`. Hãy đưa vào nhãn mà ngữ cảnh nhạc của cảnh đó cần.
- Khi đổi cảnh, bài đang phát có thể tiếp tục nếu nó vẫn nằm trong playlist mới và loại nhạc không đổi.
- Vào chiến đấu có thể lưu bài khám phá và vị trí phát của nó; khi trở lại khám phá sẽ cố khôi phục chúng. Đây là hành vi phát trực tiếp, không phải cam kết rằng tải bản lưu sẽ trở về đúng thời điểm nhạc.
- Với cùng cảnh và loại nhạc, mod nạp sau sẽ thay thế danh sách bài của mod nạp trước. Các danh sách không được ghép lại. Tệp tài nguyên trùng tên tương đối cũng có thể bị mod nạp sau ghi đè.

Dùng nhãn **Combat** cho ngữ cảnh nhạc chiến đấu. Tự đặt một tên nhãn nào đó không tạo ra ngữ cảnh mới hay thanh trượt cài đặt mới.

## Nếu im lặng

1. Kiểm tra cờ bật và âm lượng tương ứng trong **Cài đặt → Âm thanh**. Nhạc, Hiệu ứng và Âm nền độc lập với nhau.
2. Kiểm tra rằng Audio Engine và các mod nội dung cần thiết đã được nạp cho game đã chọn. Audio Engine không chứa các bài của Music Pack hay vòng lặp quán rượu của Town Pack.
3. Với nhạc, kiểm tra tên cảnh, nhãn `Ambient` hay `Combat` cần dùng, và từng tên bài. Một tham chiếu trong playlist không tạo hay tải tệp âm thanh.
4. Với âm nền, kiểm tra `defaultAmbient` và tài nguyên dưới `sounds/`. Nếu vòng lặp trước dừng lại, kiểm tra trường đó trong cảnh mới có rỗng không.
5. Kiểm tra các mod nạp sau có thể thay nhãn playlist hay tài nguyên âm thanh. Xem nhật ký game để tìm tệp âm thiếu hay lỗi phân tích playlist.

Cẩm nang này dùng đường Ogg Vorbis được hỗ trợ. Engine còn có đường nạp WAV, nhưng bộ nạp hiện tại không đọc siêu dữ liệu vùng chứa WAV; đừng cho rằng mọi WAV xuất ra đều chạy đúng. Khi phân vân, hãy chuyển nội dung mới sang Ogg Vorbis. MP3 không được bộ tạo âm thanh hiện tại hỗ trợ.
