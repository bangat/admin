<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>관리자 페이지</title>
<!-- Font Awesome CDN -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <link rel="stylesheet" href="admin.css">
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <!-- Firebase CDN -->
    <script src="https://www.gstatic.com/firebasejs/9.1.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.1.0/firebase-database-compat.js"></script>
</head>
<body>
    <header>
        <h1>관리자 페이지</h1>
    </header>

    <main>
        <select id="wardSelect">
            <option value="ward93">93 병동</option>
            <option value="ward83">83 병동</option>
            <option value="ward73">73 병동</option>
            <option value="ward71">71 병동</option>
            <option value="ward63">63 병동</option>
            <option value="ward62">62 병동</option>
            <option value="ward61">61 병동</option>
            <option value="HR">HR 병동</option>
            <option value="ward52">52 병동</option>
            <option value="ward51">51 병동</option>
            <option value="ward42">42 병동</option>
            <option value="ward41">41 병동</option>
            <option value="ICU">중환자실</option>
            <option value="dayWard">낮병동</option>
            <option value="endoscopy">내시경실</option>
            <option value="ER">응급실</option>
        </select>

        <input type="date" id="datePicker" class="date-picker" placeholder="조회 날짜를 선택하세요.">

        <div class="checkbox-container">
            <input type="checkbox" id="viewAllCheckbox">
            <label for="viewAllCheckbox">전체 날짜</label>
            <input type="checkbox" id="viewAllWardsCheckbox">
            <label for="viewAllWardsCheckbox">전체 병동</label>
            <input type="checkbox" id="laundryCheckbox">
            <label for="laundryCheckbox">세탁물</label>
        </div>

        <div class="button-container">
            <div>

<button id="cameraButton" class="nav-button"><i class="fas fa-camera"></i> 카메라</button>

                <button id="viewButton" class="nav-button">조회하기</button>


          
            </div>
            <div>
                <button class="nav-button" id="prevButton">이전</button>
                <button class="nav-button" id="nextButton">다음</button>
            </div>
        </div>

        <input type="file" id="cameraInput" accept="image/*" multiple style="display: none;">

        <section class="ward-section" id="wardSection">
            <div class="photo-gallery" id="photoGallery">
                <!-- 여기에 선택된 병동의 사진을 동적으로 추가할 예정 -->
            </div>
        </section>

    </main>

    <div id="myModal" class="modal">
        <span class="close">&times;</span>
        <div class="modal-content">
            <img id="modalImage">
        </div>
    </div>

    <footer>
    </footer>

    <script>
        // Firebase 설정 객체
        const firebaseConfig = {
            apiKey: "AIzaSyAlzPWmeLacWdnW0we7CtfnU2szvxgdIzc",
            authDomain: "hallymlinen.firebaseapp.com",
            databaseURL: "https://hallymlinen-default-rtdb.firebaseio.com",
            projectId: "hallymlinen",
            storageBucket: "hallymlinen.appspot.com",
            messagingSenderId: "867775830688",
            appId: "1:867775830688:web:cef3ed4e70ae9818f347c5",
            measurementId: "G-T02ZFK18L3"
        };

        // Firebase 초기화
        firebase.initializeApp(firebaseConfig);

        // Firebase Realtime Database 참조
        const database = firebase.database();

        const API_KEY = '197c02787b4764d219198f8c2a68656a';

        $(document).ready(function() {
            let currentPhotoIndex = 0;

            $('#viewButton').on('click', function() {
                var ward = $('#wardSelect').val();
                var date = $('#datePicker').val();
                var viewAllWards = $('#viewAllWardsCheckbox').is(':checked');
                var isLaundry = $('#laundryCheckbox').is(':checked');

                if ($('#viewAllCheckbox').is(':checked')) {
                    loadPhotos(ward, null, viewAllWards, isLaundry); // 전체 보기
                } else if (ward && date) {
                    loadPhotos(ward, date, viewAllWards, isLaundry);
                } else {
                    alert('병동과 날짜를 선택하세요.');
                }
            });

            $('#wardSelect').on('change', function() {
                var ward = $(this).val();
                var date = $('#datePicker').val();
                if (ward && date) {
                    loadPhotos(ward, date, $('#viewAllWardsCheckbox').is(':checked'), $('#laundryCheckbox').is(':checked'));
                }
            });

            $('#datePicker').on('change', function() {
                var ward = $('#wardSelect').val();
                var date = $(this).val();
                if (ward && date) {
                    loadPhotos(ward, date, $('#viewAllWardsCheckbox').is(':checked'), $('#laundryCheckbox').is(':checked'));
                }
            });

           $('#cameraButton').on('click', function() {
    if ($('#wardSelect').val()) {
        $('#cameraInput').click();  // 'capture' 속성 삭제
    } else {
        alert('병동을 선택하세요.');
    }
});

            $('#cameraInput').on('change', function() {
                var files = this.files;
                if (files.length > 0) {
                    Array.from(files).forEach(file => {
                        var reader = new FileReader();
                        reader.onload = function(e) {
                            var img = new Image();
                            img.src = e.target.result;
                            img.onload = function() {
                                if (confirm('세탁물로 분류하시겠습니까?')) {
                                    uploadPhoto(file, true);
                                } else {
                                    uploadPhoto(file, false);
                                }
                            };
                        };
                        reader.readAsDataURL(file);
                    });
                }
            });

            $('#prevButton').on('click', function() {
                if (currentPhotoIndex > 0) {
                    currentPhotoIndex--;
                    showPhoto(currentPhotoIndex);
                }
            });

            $('#nextButton').on('click', function() {
                let photos = $('.photo-container');
                if (currentPhotoIndex < photos.length - 1) {
                    currentPhotoIndex++;
                    showPhoto(currentPhotoIndex);
                }
            });

            // 터치 이벤트 핸들러 추가
            $('#photoGallery').on('touchstart', function(event) {
                startX = event.originalEvent.touches[0].clientX;
                startY = event.originalEvent.touches[0].clientY;
            });

            $('#photoGallery').on('touchmove', function(event) {
                endX = event.originalEvent.touches[0].clientX;
                endY = event.originalEvent.touches[0].clientY;
            });

            $('#photoGallery').on('touchend', function() {
                let deltaX = endX - startX;
                let deltaY = endY - startY;

                // 좌우 스와이프 거리 차이가 상하 스와이프 거리 차이보다 크다면 좌우 스와이프로 간주
                if (Math.abs(deltaX) > Math.abs(deltaY)) {
                    if (deltaX > 50) {
                        // 오른쪽 스와이프 -> 이전 사진
                        if (currentPhotoIndex > 0) {
                            currentPhotoIndex--;
                            showPhoto(currentPhotoIndex);
                        }
                    } else if (deltaX < -50) {
                        // 왼쪽 스와이프 -> 다음 사진
                        let photos = $('.photo-container');
                        if (currentPhotoIndex < photos.length - 1) {
                            currentPhotoIndex++;
                            showPhoto(currentPhotoIndex);
                        }
                    }
                }
            });

            function loadPhotos(selectedWard, date, viewAllWards, isLaundry) {
                let wards = viewAllWards ? getAllWards() : [selectedWard];
                let photos = [];

                wards.forEach(ward => {
                    let wardPhotosRef = isLaundry ? database.ref('laundryPhotos/' + ward) : database.ref('photos/' + ward);
                    wardPhotosRef.once('value', (snapshot) => {
                        snapshot.forEach((childSnapshot) => {
                            let photo = childSnapshot.val();
                            photo.ward = ward;
                            photos.push(photo);
                        });

                        // 데이터를 모두 가져온 후에 화면에 표시
                        if (ward === wards[wards.length - 1]) {
                            displayPhotos(photos, date);
                        }
                    });
                });
            }

            function displayPhotos(photos, date) {
                $('#photoGallery').empty();

                let filteredPhotos = date ? photos.filter(photo => photo.date.split(' ')[0] === date) : photos;

                filteredPhotos.forEach(photo => {
                    let photoContainer = $('<div>').addClass('photo-container');
                    let formattedDate = formatDateToKorean(photo.date);
                    let dateLabel = $('<div>').addClass('date-label').text(formattedDate);
                    let wardLabel = $('<div>').addClass('ward-label').text(getWardLabel(photo.ward));
                    let img = $('<img>').attr('src', photo.url);
                    let deleteButton = $('<button>').addClass('delete-button').text('삭제하기');

                    deleteButton.on('click', function() {
                        if (confirm('정말 삭제하시겠습니까?')) {
                            deletePhoto(photo.ward, photo);
                        }
                    });

                    img.on('click', function() {
                        openModal(photo.url);
                    });

                    photoContainer.append(dateLabel, img, wardLabel, deleteButton);
                    $('#photoGallery').append(photoContainer);
                });

                currentPhotoIndex = 0;
                showPhoto(currentPhotoIndex);
                toggleNavigationButtons(filteredPhotos.length);
            }

            function showPhoto(index) {
                let photos = $('.photo-container');
                photos.hide();
                if (photos.length > 0) {
                    $(photos[index]).show();
                }
            }

            function toggleNavigationButtons(photoCount) {
                if (photoCount <= 1) {
                    $('#prevButton, #nextButton').hide();
                } else {
                    $('#prevButton, #nextButton').show();
                }
            }

            function uploadPhoto(file, isLaundry) {
                let ward = $('#wardSelect').val();
                let date = new Date(new Date().getTime() + (9 * 60 * 60 * 1000)).toISOString().slice(0, 16).replace("T", " ");

                let formData = new FormData();
                formData.append('image', file);
                formData.append('key', API_KEY);

                // 업로드 버튼을 비활성화하고 업로드 중 메시지로 변경
                $('#cameraButton').prop('disabled', true).text('업로드 중...');

                $.ajax({
                    url: 'https://api.imgbb.com/1/upload',
                    type: 'POST',
                    data: formData,
                    contentType: false,
                    processData: false,
                    success: function(response) {
                        let photoUrl = response.data.url;
                        let refPath = isLaundry ? 'laundryPhotos/' + ward : 'photos/' + ward;
                        let newPhotoRef = database.ref(refPath).push();
                        newPhotoRef.set({
                            url: photoUrl,
                            date: date
                        }).then(() => {
                            // 사진 업로드가 완료되면 업로드 버튼을 다시 활성화하고 텍스트를 "업로드"로 변경
                            $('#cameraButton').prop('disabled', false).text('카메라');
                            
                            // 사진 로드 함수 호출
                            loadPhotos(ward, date.split(' ')[0], $('#viewAllWardsCheckbox').is(':checked'), isLaundry);
                        });
                    },
                    error: function() {
                        // 업로드 실패 시 처리
                        alert('사진 업로드 중 오류가 발생했습니다.');
                        
                        // 업로드 버튼을 다시 활성화하고 텍스트를 "업로드"로 변경
                        $('#cameraButton').prop('disabled', false).text('카메라');
                    }
                });
            }

            function deletePhoto(ward, photo) {
    let refPath = $('#laundryCheckbox').is(':checked') ? 'laundryPhotos/' + ward : 'photos/' + ward;
    let photoRef = database.ref(refPath).orderByChild('url').equalTo(photo.url);
    photoRef.once('value', function(snapshot) {
        snapshot.forEach(function(childSnapshot) {
            childSnapshot.ref.remove().then(() => {
                alert('사진이 삭제되었습니다.');
                loadPhotos(ward, photo.date.split(' ')[0], $('#viewAllWardsCheckbox').is(':checked'), $('#laundryCheckbox').is(':checked'));
            }).catch((error) => {
                console.error('삭제 중 오류가 발생했습니다:', error);
                alert('삭제 중 오류가 발생했습니다.');
            });
        });
    });
}

            function openModal(imageUrl) {
                var modal = document.getElementById("myModal");
                var modalImg = document.getElementById("modalImage");

                modal.style.display = "block";
                modalImg.src = imageUrl;

                var span = document.getElementsByClassName("close")[0];
                span.onclick = function() {
                    modal.style.display = "none";
                }
            }

            function formatDateToKorean(dateString) {
                var date = new Date(dateString);
                var year = ('' + date.getFullYear()); // .slice(-2); // 년도를 두 자리 숫자로 표시
                var month = ('0' + (date.getMonth() + 1)).slice(-2);
                var day = ('0' + date.getDate()).slice(-2);
                var hours = date.getHours();
                var minutes = ('0' + date.getMinutes()).slice(-2);
                var period = hours >= 12 ? '오후' : '오전'; // 오후/오전 구분

                // 12시간제로 변경 및 오전/오후 표시
                if (hours === 0) {
                    hours = 12; // 자정을 12시로 표시
                } else if (hours > 12) {
                    hours -= 12; // 오후 시간을 12시간제로 변환
                }

                return `${year}-${month}-${day} ${period}${hours}시${minutes}분`;
            }

            function getWardLabel(wardCode) {
                var wardLabels = {
                    'ward93': '93 병동',
                    'ward83': '83 병동',
                    'ward73': '73 병동',
                    'ward71': '71 병동',
                    'ward63': '63 병동',
                    'ward62': '62 병동',
                    'ward61': '61 병동',
                    'HR': 'HR 병동',
                    'ward52': '52 병동',
                    'ward51': '51 병동',
                    'ward42': '42 병동',
                    'ward41': '41 병동',
                    'ICU': '중환자실',
                    'dayWard': '낮병동',
                    'endoscopy': '내시경실',
                    'ER': '응급실'
                };
                return wardLabels[wardCode] || wardCode;
            }

              function getAllWards() {
                return [
                    "ward93", "ward83", "ward73", "ward71", "ward63",
                    "ward62", "ward61", "HR", "ward52", "ward51",
                    "ward42", "ward41", "ICU", "dayWard", "endoscopy", "ER"
                ];
            }

            // 초기 로드 시 현재 날짜를 한국 시간으로 설정
            let today = new Date();
            today.setHours(today.getHours() + 9); // 한국 시간 (UTC+9)으로 조정
            $('#datePicker').val(today.toISOString().slice(0, 10));

            // "관리자 페이지" 클릭 시 index.html로 이동
            $('header h1').on('click', function() {
                window.location.href = 'index.html';
            });
        });
    </script>
</body>
</html>
